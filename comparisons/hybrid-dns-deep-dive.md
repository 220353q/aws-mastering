# Hybrid DNS Deep Dive

> Hybrid DNSは「オンプレミスとAWSの名前解決をつなぐ」だけではない。**誰がQueryを送り、どのResolverが受け、どのDomainをどこへForwardし、どの経路で返すか**を設計する。

SAP-C02では、Direct ConnectやVPNが接続済みでも、DNSが成立していないためApplicationが通信できないケースが出る。IP到達性と名前解決を分けて考える。

```text
Connectivity: Packetが相手へ届くか
DNS: Nameを接続先情報へ変換できるか
```

---

# 1. DNS Queryの基本

```text
Application
  → OS stub resolver
  → configured recursive resolver
  → authoritative / forwarder
  → answer
  → Application connects to returned target
```

DNS回答を得ても、実際の通信経路・Security Group・Firewallが成立するとは限らない。

---

# 2. VPC DNSの基本

VPC内Resourceは、VPC Resolverを利用して次を解決できる。

- AWS提供のPrivate DNS
- Private Hosted Zone
- Public DNS
- Resolver ruleで指定した外部Domain

VPCのDNS attributesも確認する。

- DNS resolution support
- DNS hostnames

これらを無効化すると、期待する名前解決ができない場合がある。

---

# 3. Route 53 Resolver Endpoint

## Inbound Endpoint

オンプレミス側からAWS側の名前を問い合わせる入口。

```text
On-prem client
  → On-prem DNS
  → Conditional forward
  → Resolver Inbound Endpoint IP
  → Private Hosted Zone / AWS private DNS
  → Answer
```

### 一言

**オンプレミスからAWSへQueryを入れる入口。**

## Outbound Endpoint

AWS側からオンプレミス側の名前を問い合わせる出口。

```text
EC2 / ECS / Lambda in VPC
  → VPC Resolver
  → Resolver rule
  → Outbound Endpoint
  → On-prem DNS
  → Answer
```

### 一言

**AWSからオンプレミスDNSへQueryを出す出口。**

## 混同しない

| Endpoint | Queryの向き | 主な用途 |
|---|---|---|
| Inbound | On-prem → AWS | Private Hosted Zoneをオンプレから解決 |
| Outbound | AWS → On-prem | 社内DomainをAWSから解決 |

InboundとOutboundは、Application Trafficの入口・出口ではない。DNS Query専用の役割である。

---

# 4. Resolver Rule

特定DomainのQueryを指定DNS serverへForwardする。

```text
corp.example.com
  → 10.0.0.10, 10.0.1.10
```

## Ruleの要素

- Domain name
- Target IP addresses
- Associated VPCs
- Outbound Endpoint

## 最長一致

より具体的なDomain ruleが優先されるように設計する。

```text
example.com
corp.example.com
```

`corp.example.com`のQueryは、より具体的なRuleへ送る。

---

# 5. Conditional Forwarding

特定Domainだけ別Resolverへ送る。

```text
On-prem DNS
  ├─ aws.internal → Resolver Inbound Endpoint
  └─ other names  → existing resolver path
```

すべてのQueryをAWSへ送らない。Domain所有者とAuthoritative sourceを明確にする。

---

# 6. Private Hosted Zone

Public Internetへ公開せず、関連付けたVPC内から解決するZone。

例:

```text
db.prod.internal
  → private ALB / RDS endpoint / private IP
```

## 注意

- VPC associationが必要
- Cross-account associationを設計する
- On-premから直接見えるわけではない
- Resolver Inbound EndpointとForwardingが必要

Private Hosted ZoneはNetwork connectivityを作らない。

---

# 7. Split-horizon DNS

同じDomain nameに、問い合わせ元によって異なるAnswerを返す。

```text
Public DNS:
app.example.com → CloudFront

Private DNS:
app.example.com → internal ALB
```

## 用途

- 社内UserはPrivate path
- Internet UserはPublic path
- Migration中に環境別Route

## Risk

- 調査時に人によってAnswerが違う
- Cacheの影響
- CertificateとHost name
- Internal / external設定のDrift

`dig`や`nslookup`を実行した場所と利用Resolverを記録する。

---

# 8. Centralized DNS Architecture

複数Account・VPCでResolver Endpointを共有する場合、Network accountまたはShared Services accountへ集約する。

```text
On-prem DNS
  ↔ Central DNS VPC
       ├─ Inbound Endpoint
       ├─ Outbound Endpoint
       └─ Resolver Rules
            ↓ shared / associated
          Workload VPCs
```

## 利点

- Endpoint数を減らす
- Ruleを集中管理
- On-prem DNS targetを標準化

## 注意

- Resolver rule association
- AWS RAMによる共有
- Network path
- Security Group
- Availability Zone分散
- Central VPC障害影響

Centralizeしても、すべての名前を一つのTeamが無秩序に管理しない。Domain delegationとOwnershipを決める。

---

# 9. DNSとTransit Gateway

Transit GatewayはPacket routingを提供する。DNS QueryのForwarding ruleやHosted Zone associationを自動作成しない。

```text
TGW available
  ≠ DNS automatically integrated
```

必要なもの:

- VPC route
- TGW route
- Return route
- DNS Resolver Endpoint
- Resolver rule
- Security rules

---

# 10. DNSとDirect Connect / VPN

DXやVPNがあっても、DNS serverへ到達できるRouteが必要。

## 確認

```text
AWS resource
  → VPC route table
  → TGW / VGW
  → DX / VPN
  → On-prem firewall
  → DNS server
  → return path
```

DNSは通常UDP/TCP 53を使う。大きなResponseやZone transferなどではTCPも考える。

---

# 11. DNS Firewall

Route 53 Resolver DNS Firewallを使い、Domain listに基づいてDNS Queryを許可・拒否・Alertできる。

用途:

- Known malicious domainのBlock
- Exfiltration対策
- Organization標準Rule

Network FirewallやWAFの代替ではない。DNS query layerのControlである。

---

# 12. DNSSEC

DNS応答が改ざんされていないことを暗号学的に検証する仕組み。

## 分ける

- Public Hosted Zone signing
- Resolver-side validation

DNSSECは通信内容を暗号化しない。TLSの代替ではない。

---

# 13. Endpoint Private DNS

Interface VPC EndpointでPrivate DNSを有効にすると、AWS Serviceの通常Public hostnameをVPC内からPrivate endpoint IPへ解決できる場合がある。

```text
service.region.amazonaws.com
  → Private IP of Interface Endpoint
```

## 注意

- VPC DNS設定
- Private DNSの対応Service
- On-premから利用する場合のResolver経路
- Centralized Endpoint VPCのDNS設計

Gateway EndpointはRoute tableを使うため、Interface Endpointと同じDNS動作ではない。

---

# 14. DNS CacheとTTL

DNS切替は即時とは限らない。

```text
Authoritative record update
  → Recursive resolver cache expires
  → Client cache expires
  → new answer used
```

## 切替前

- TTLを事前に下げる
- Client / JVM / OS cacheを確認
- Connection poolingを確認

## 切替後

- TTLを戻す
- Old endpointへのTrafficを観測
- Long-lived connectionを確認

Route 53 Failoverを設定しても、既存Connectionが自動的に新Endpointへ移動するとは限らない。

---

# 15. DNS Failover

Route 53 Health CheckとFailover recordを使う。

```text
Primary healthy
  → primary answer
Primary unhealthy
  → secondary answer
```

## 注意

- Private endpointのHealth Check方法
- Application healthとEndpoint health
- TTL
- False positive
- Data layer readiness
- SecondaryのCapacity

DNSだけ切り替えても、Secondary DBが書き込み可能でなければBusiness continuityは成立しない。

---

# 16. DNS Routing Policies

| Policy | 主な判断 |
|---|---|
| Simple | 単純なAnswer |
| Weighted | 比率配分、移行、Canary |
| Latency | AWS Region間で低Latency候補 |
| Failover | Primary / Secondary |
| Geolocation | User locationに基づく |
| Geoproximity | Resource locationとBias |
| Multivalue answer | 複数healthy record |

DNS routingとLoad Balancer routingを分ける。

- DNS: Clientが接続先を得る前
- ALB: RequestがALBへ届いた後

---

# 17. Overlapping Domain

複数のPrivate Hosted ZoneやResolver ruleで似たDomainを管理すると、意図しないAnswerになる。

例:

```text
PHZ: example.com
Rule: corp.example.com → on-prem
```

Domain ownershipを一覧化し、最長一致とAssociationを確認する。

---

# 18. Overlapping CIDRとDNS

PrivateLinkはCIDR重複環境でもService単位接続に使いやすいが、DNS名からInterface Endpointへ解決する仕組みを設計する必要がある。

VPC Peering / TGWでCIDR重複が問題になる場合でも、名前解決だけ成功してもPacket routingは成立しない。

---

# 19. Troubleshooting Flow

## Step 1: 名前は何か

```text
Expected FQDN:
Actual FQDN:
Search suffix:
```

## Step 2: 誰へQueryしているか

- `/etc/resolv.conf`
- Windows DNS settings
- Container DNS settings
- VPC DHCP options
- On-prem resolver

## Step 3: Answerを確認

```bash
dig app.internal
nslookup app.internal
```

確認:

- Answer
- Authority
- TTL
- Resolver server
- CNAME chain

## Step 4: RuleとZone

- Resolver rule match
- VPC association
- Private Hosted Zone
- Conditional forwarder
- RAM share

## Step 5: Endpoint

- Inbound / Outbound Endpoint status
- IP addresses
- AZ
- Security Group

## Step 6: Network

- Route table
- TGW route
- DX / VPN
- NACL
- Firewall
- Return path

## Step 7: Application connection

- Returned IPへ到達できるか
- Port
- TLS hostname
- Certificate
- Application timeout

---

# 20. Flow Logsで分かること・分からないこと

VPC Flow Logsでは、Network interfaceを通るIP trafficを確認できる。

分かる:

- Source / destination
- Port
- Accept / Reject
- Bytes / packets

分からない:

- DNS Query nameの詳細
- Application error
- Resolver rule selection理由

DNS query logsと組み合わせる。

---

# 21. Route 53 Resolver Query Logging

DNS Queryを記録し、次を調査する。

- 誰がどのDomainをQueryしたか
- Query type
- Response code
- Unusual domain

用途:

- Troubleshooting
- Security monitoring
- Migration前の依存関係調査

Log volumeとRetentionを設計する。

---

# 22. Reachability Analyzerとの役割分担

Reachability AnalyzerはNetwork configurationを解析して到達可能性を調べる。DNSの正しいAnswerを保証しない。

```text
DNS test: Name → IP
Reachability test: Source → returned IP:port
```

両方を順に確認する。

---

# 23. よくある構成

## On-premからPrivate ALB

```text
On-prem client
  → On-prem DNS
  → forward internal.aws.example
  → Inbound Endpoint
  → Private Hosted Zone
  → Internal ALB
```

## AWSからOn-prem DB

```text
ECS task
  → VPC Resolver
  → Rule db.corp.example
  → Outbound Endpoint
  → On-prem DNS
  → DB IP
  → DX/VPN path
```

## Shared DNS

```text
Workload VPC
  → shared Resolver rule
  → Central DNS VPC Outbound Endpoint
  → On-prem DNS
```

---

# 24. Anti-patterns

- `/etc/hosts`を多数Serverへ配布
- IP addressをApplicationへHard-code
- 全Domainを一律Forward
- InboundとOutboundを逆に覚える
- TGW作成だけでDNS統合済みと考える
- Public Hosted ZoneへInternal IPを無計画に登録
- TTLを下げずに大規模Cutover
- DNS成功だけでApplication接続成功と判断
- Single Resolver Endpoint IPだけに依存
- Query loggingなしで障害原因を推測

---

# 25. SAP-C02での選び方

## On-premからAWS Private Hosted Zoneを解決

On-prem DNSのConditional Forwarder + Resolver Inbound Endpoint。

## AWSから社内Domainを解決

Resolver Outbound Endpoint + Forwarding Rule。

## 複数Accountへ同じRuleを展開

Central DNS設計 + AWS RAM / Association。

## VPC全体を接続せずServiceだけ提供

PrivateLink。DNS名とEndpointの設計も含める。

## Region切替

Route 53 routing + Health + TTL + Data readiness。

## DNS QueryをSecurity control

Resolver DNS Firewall + Query logging。

---

# 26. 設計テンプレート

```text
Client location:
Query name:
Authoritative owner:
Recursive resolver:
Forwarding rule:
Inbound / Outbound Endpoint:
Network path:
Return path:
Security rules:
TTL:
Logging:
Failure behavior:
Ownership:
```

## 一文の完成形

> オンプレミスのClientがAWS Private Hosted Zoneを解決する必要があるため、オンプレDNSに対象DomainのConditional Forwarderを設定し、複数AZのRoute 53 Resolver Inbound EndpointへQueryを送る。DXまたはVPNのRouteとUDP/TCP 53を許可し、Query logで利用状況を確認する。Transit GatewayだけではDNS Forwardingは構成されない。

## 関連資料

- [Route 53](../services/networking/route53.md)
- [Amazon VPC](../services/networking/vpc.md)
- [Transit Gateway](../services/networking/transitgateway.md)
- [Direct Connect](../services/networking/direct-connect.md)
- [Site-to-Site VPN](../services/networking/site-to-site-vpn.md)
- [PrivateLink](../services/networking/privatelink.md)
- [Networking Foundations Deep Dive](networking-foundations-deep-dive.md)
