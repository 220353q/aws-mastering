# AWS Edge Security Comparison

> WAF、Shield、CloudFront、Global Accelerator、Network Firewall、Security Groupは、すべて「通信を守る」ように見える。しかし、**どのLayerで、どのTrafficを、どの位置で、どの情報を見て制御するか**が異なる。

## 最初の判断

```text
HTTP(S) request内容を検査       → AWS WAF
DDoS吸収・保護                  → AWS Shield
HTTP(S)をEdge cache / proxy      → CloudFront
TCP/UDPをGlobal Anycastで入口化  → Global Accelerator
VPC境界でStateful inspection     → AWS Network Firewall
Instance / ENI単位の許可         → Security Group
Subnet単位のStateless rule       → Network ACL
Appliance fleetを挿入            → Gateway Load Balancer
```

---

## 比較表

| Service | 主Layer | 配置 | 見る情報 | 主目的 |
|---|---|---|---|---|
| CloudFront | L7 HTTP(S) | Global edge | URL、Header、Cache key | CDN、Origin protection |
| AWS WAF | L7 HTTP(S) | CloudFront / ALB / API Gateway等 | URI、Header、Body一部、IP | Web attack control |
| Shield Standard | L3/L4中心 | Eligible resources | Traffic pattern | Basic DDoS protection |
| Shield Advanced | L3-L7支援 | Protected resources | Attack telemetry | Enhanced DDoS response |
| Global Accelerator | L4 | Global edge | Endpoint health、TCP/UDP flow | Anycast、Global routing |
| Network Firewall | L3-L7 network inspection | VPC routing path | IP、Port、Protocol、domain/signature等 | VPC egress/ingress inspection |
| Security Group | L3/L4 | ENI / Resource | IP、Port、Protocol | Stateful allow control |
| Network ACL | L3/L4 | Subnet | IP、Port、Protocol | Stateless allow/deny |
| GWLB | L3/L4 service insertion | VPC path | Encapsulated traffic | Appliance scaling |

---

# 1. CloudFront

## 役割

Viewer requestをEdgeで受け、Cache hitならOriginへ行かずResponseを返す。MissならOriginへProxyする。

```text
Viewer
  → CloudFront edge
      ├─ cache hit → response
      └─ miss → origin
```

## Security価値

- Originを直接公開しない設計
- WAF integration
- Shield integration
- TLS termination
- Geo restriction
- Signed URL / cookie
- S3 OAC

CloudFront自体を細粒度AuthorizationやVPC Firewallの代替にしない。

---

# 2. AWS WAF

## 役割

HTTP(S) requestのApplication layer情報をRuleで評価する。

## Rule例

- IP set
- Rate-based rule
- SQL injection pattern
- XSS pattern
- URI path
- Header
- Geographic match
- Managed rule group

## 選ぶ条件

- Web application attack
- Bot / rate control
- Path-specific rule
- HTTP request inspection

## 選ばない条件

- Arbitrary TCP/UDP
- VPC east-west全Traffic
- OS vulnerability scan
- DDoS容量吸収だけ

---

# 3. Shield

## Shield Standard

AWSの対象ServiceへBasic DDoS protectionを提供する。

## Shield Advanced

Enhanced protection、Visibility、Response支援、Cost protection特性等を要件に応じて利用する。

## WAFとの違い

- Shield: DDoS protection
- WAF: HTTP request rule control

組み合わせる。

```text
Internet
  → Shield
  → CloudFront
  → WAF
  → ALB / API Gateway
```

順序図は論理的な役割表現であり、内部実装順序を単純化している。

---

# 4. Global Accelerator

## 役割

Static Anycast IPを提供し、AWS Global Networkを使ってHealthy regional endpointへTCP/UDP flowを送る。

## 選ぶ条件

- Static global IP
- TCP / UDP
- Multi-Region failover
- Global network path
- Non-cacheable traffic

## CloudFrontとの違い

| 観点 | CloudFront | Global Accelerator |
|---|---|---|
| Protocol | HTTP(S)中心 | TCP / UDP |
| Cache | あり | なし |
| Static Anycast IP | 主目的ではない | 主機能 |
| Content delivery | 強い | Packet routing |
| Origin | HTTP origin | ALB、NLB、EC2、EIP等 |

---

# 5. Network Firewall

## 役割

VPC routeへFirewall endpointを挿入し、Stateful / Stateless ruleでTrafficをInspectionする。

## Use case

- Centralized egress inspection
- Domain filtering
- Protocol / signature inspection
- Inter-VPC inspection
- Ingress inspection architecture

## 設計

- Firewall subnet
- Route symmetry
- AZ mapping
- TGW integration
- Rule group
- Logging
- Fail-open / fail-closed要件

## WAFとの違い

WAFはWeb request。Network FirewallはVPC network path。

---

# 6. Security Group

## 役割

Resource / ENIへ入出力を許可するStateful firewall。

```text
Inbound allow request
  → return traffic automatically allowed
```

## 特徴

- Allow rules
- Stateful
- Resource reference
- ENI scope

## 設計原則

- Internet CIDRよりSecurity Group referenceを優先できる場面
- Tier間を明示
- Outboundも要件に合わせる
- Shared SGの影響範囲を理解

---

# 7. Network ACL

## 役割

Subnet boundaryのStateless allow/deny。

## 特徴

- Ordered rule number
- Allow and deny
- Inbound / outbound両方
- Ephemeral ports

Security Groupより粗いSubnet control。SGの代替ではなくDefense in depthや明示Deny等で利用する。

---

# 8. Gateway Load Balancer

## 役割

Firewall、IDS/IPS等のVirtual appliance fleetを透過的にScale・挿入する。

```text
Traffic
  → GWLB Endpoint
  → GWLB
  → Appliance fleet
  → return path
```

## 選ぶ条件

- Third-party appliance
- Transparent insertion
- Appliance scaling
- Central inspection VPC

Network FirewallはAWS-managed firewall。GWLBはAppliance platform。

---

# 9. ALB / NLBとの関係

## ALB

L7 load balancing、Host / Path routing、WAF integration。

## NLB

L4、high performance、static IP特性、TCP/UDP/TLS。

## Security選定

- WAFが必要 → ALB / CloudFront / API Gateway等対応Resource
- Source IP / L4 → NLB要件を評価
- Appliance → GWLB

Load BalancerはFirewallそのものではない。

---

# 10. API Gateway

API公開で次を提供する。

- Authentication integration
- Authorization
- Throttling
- Usage plan
- Request validation
- WAF integration

ALBよりAPI managementが強い。単なる内部Scheduler→ECSには不要な場合がある。

---

# 11. Origin Protection

CloudFrontを置いてもOriginがInternetから直接到達可能なら、WAFやCacheを迂回され得る。

## S3

OAC + Bucket policy。

## ALB / Custom origin

- Secret Header検証
- Managed prefix list / Network control
- CloudFront VPC origin等、利用可能な機能を要件で確認
- Origin TLS

EdgeとOriginの両方を設計する。

---

# 12. Rate limiting

## WAF rate-based rule

IP等を基準にWeb request rateを制御。

## API Gateway throttling

API stage / method / usage plan等でrequest rateとburstを制御。

## Application rate limit

User、Tenant、Business keyで制御。

同じ「rate limit」でも識別単位が違う。

---

# 13. Authentication vs Edge Security

- TLS: Communication encryption / server identity
- Authentication: User / client identity
- Authorization: Allowed action
- WAF: Malicious request pattern control
- Shield: DDoS

WAF ruleをUser authorizationの代替にしない。

---

# 14. Ingress Pattern

## Public Web

```text
User
  → Route 53
  → CloudFront
  → WAF / Shield
  → ALB
  → Application
```

## Global TCP

```text
Client
  → Global Accelerator
  → NLB
  → Service
```

## Central appliance

```text
VPC / Internet traffic
  → GWLB Endpoint
  → Appliance VPC
  → Firewall fleet
```

## VPC egress

```text
Workload subnet
  → Network Firewall endpoint
  → NAT / TGW / Internet
```

Route symmetryを確認する。

---

# 15. Logging

| Service | 主なLog / telemetry |
|---|---|
| CloudFront | Access logs / real-time logs等 |
| WAF | Web ACL logs、sampled requests |
| Shield | Attack events / metrics |
| Network Firewall | Flow / alert logs |
| ALB | Access logs |
| VPC | Flow Logs |
| Route 53 | Query logs / health |

Logを中央Accountへ集約し、Retention、Cost、Accessを設計する。

---

# 16. DDoS Response

## Before

- Architecture protection
- Auto Scaling
- WAF rules
- Shield protection
- Origin capacity
- Runbook
- Contact path

## During

- Identify target
- Check Shield / WAF metrics
- Apply rate / block rule carefully
- Protect origin
- Scale
- Preserve logs

## After

- Analyze pattern
- Rule tuning
- Cost review
- Game day

DDoS対策を一つのServiceだけに依存しない。

---

# 17. SAP-C02選択表

| 問題文 | 候補 |
|---|---|
| SQLi / XSSをWeb入口で防ぐ | WAF |
| DDoS enhanced response | Shield Advanced |
| Global static IP for TCP | Global Accelerator |
| Static content cache + origin protection | CloudFront |
| VPC egress domain / signature inspection | Network Firewall |
| Existing virtual firewall fleet | GWLB |
| EC2 tier間Port許可 | Security Group |
| Subnet単位で明示Deny | NACL |
| APIごとのthrottle / auth | API Gateway |

---

# 18. よくある誤答

- WAFで任意TCP/UDPを検査する
- ShieldでSQL injection ruleを書く
- CloudFrontをVPC Firewallとして使う
- Global AcceleratorがContentをCacheする
- Security GroupにDeny ruleを書く
- NACLがStatefulなのでReturn rule不要
- Network FirewallをApplication user認証に使う
- GWLB自体がFirewall signatureを持つ
- CloudFrontを置くだけでOrigin bypass不可になる
- WAFを有効化すればApplication vulnerabilityが修正される

---

# 19. 設計テンプレート

```text
Protocol:
Public / private:
Client locations:
Protected resource:
Threat:
Layer:
Cache needed:
Static IP needed:
Inspection content:
Rate identity:
Origin restriction:
Route symmetry:
Logging:
Failover:
Cost:
```

## 完成した説明

> 世界向けHTTP APIでSQL injectionとBotによる大量Requestを制御するため、CloudFront前段にWAFを関連付け、Managed ruleとRate-based ruleを適用する。ShieldはDDoS保護として併用するが、SQL injectionのRequest内容を判定するServiceではない。Origin ALBはCloudFront経由に制限し、WAF logとALB access logをSecurity accountへ集約する。

## 関連資料

- [CloudFront](../services/networking/cloudfront.md)
- [Global Accelerator](../services/networking/global-accelerator.md)
- [Elastic Load Balancing](../services/networking/elb.md)
- [WAF](../services/security/waf.md)
- [Shield](../services/security/shield.md)
- [Network Firewall](../services/security/network-firewall.md)
- [ACM](../services/security/acm.md)
