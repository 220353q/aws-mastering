# AWS Network Firewall

## Positioning

AWS Network Firewall は、VPC境界でネットワークトラフィックを検査・制御するマネージドネットワークファイアウォール。SAP-C02では、Security Group、NACL、WAF、GWLB + third-party applianceとの違いが問われる。

---

## Core Concepts

| 概念 | 内容 |
|---|---|
| Firewall | VPC内に配置するファイアウォールエンドポイント群 |
| Firewall Policy | Stateless / Stateful rule groupsをまとめるポリシー |
| Stateless Rules | 各パケットを独立評価。NACLに近い考え方 |
| Stateful Rules | フロー状態を見て評価。Suricata互換ルールも利用可能 |
| Rule Group | 再利用可能なルール集合 |
| Logging | CloudWatch Logs / S3 / Kinesis Data Firehose等へログ出力 |

---

## Stateless vs Stateful

| 種類 | 特徴 | 近い概念 |
|---|---|---|
| Stateless | パケット単体で評価。高速・単純 | NACL |
| Stateful | 接続状態やアプリ層情報を踏まえて評価 | Security Group / IDS/IPS |

厳密にはSG/NACLと同じではないが、試験では「どの粒度で制御するか」の理解が重要。

---

## Typical Centralized Inspection Pattern

```text
Spoke VPCs
  → Transit Gateway
  → Inspection VPC
    → AWS Network Firewall endpoints
  → Egress / Shared Services / On-prem
```

複数VPCのアウトバウンド通信、東西トラフィック、オンプレ接続を中央で検査する構成で出題されやすい。

---

## Network Firewall vs WAF vs Security Group

| 要件 | 選択 |
|---|---|
| HTTPのSQLi/XSS/Bot対策 | WAF |
| VPC境界のL3-L7ネットワーク検査 | Network Firewall |
| EC2/ENI単位の許可制御 | Security Group |
| サブネット単位のステートレス制御 | NACL |
| サードパーティアプライアンスを挿入 | Gateway Load Balancer |

---

## Use Cases

- Egress filtering: インターネット宛通信の制御
- Centralized inspection: 複数VPC通信の集約検査
- Intrusion prevention: Suricata互換ルールで脅威検出/遮断
- Domain filtering: 許可/拒否ドメイン制御
- Compliance: ネットワーク境界での監査ログ取得

---

## SAP-C02 Focus

Network Firewallを選ぶキーワード:

- inspect traffic between VPCs / subnets / internet
- centralized network inspection
- egress filtering
- stateful inspection at VPC boundary
- Suricata-compatible rules

---

## Exam Traps

- WAFはHTTP(S)のL7 Web保護。Network FirewallはVPC内/境界のネットワーク検査。
- Security Groupだけでは高度なL7/ドメイン/IPS的制御は難しい。
- ルートテーブル設計が重要。Firewall endpointを通るように経路制御しなければ検査されない。
- 既存サードパーティFW製品を使いたい場合はGWLBの方が適切なことがある。

---

## Related

- [AWS WAF](waf.md)
- [Elastic Load Balancing - GWLB](../networking/elb.md)
- [Amazon VPC](../networking/vpc.md)
- [AWS Transit Gateway](../networking/transitgateway.md)
- [Edge Security Comparison](../../comparisons/edge-security.md)

## Official Docs

- https://docs.aws.amazon.com/network-firewall/latest/developerguide/firewall-rules-engines.html
- https://docs.aws.amazon.com/network-firewall/latest/developerguide/stateful-rule-group-options.html
