# AWS PrivateLink / VPC Endpoints — SAP-C02重点ノート

## Overview
AWS PrivateLink は、VPCからAWSサービス、他アカウントのサービス、SaaSサービスへ、プライベートIPで接続する仕組み。SAP-C02では、**Interface Endpoint / Gateway Endpoint / Endpoint Policy / PrivateLinkによるサービス公開** の違いが頻出。

---

## VPC Endpointの種類

| 種類 | 対象 | 仕組み | コスト | 代表例 |
|---|---|---|---|---|
| Gateway Endpoint | S3 / DynamoDB | ルートテーブルにprefix listを追加 | 追加料金なし | VPC内EC2からS3へNAT不要アクセス |
| Interface Endpoint | 多くのAWSサービス / PrivateLink | ENI + Private IP | 時間課金/データ処理課金 | ECR, KMS, Secrets Manager, SSM等 |
| Gateway Load Balancer Endpoint | セキュリティアプライアンス | GWLB経由 | 課金あり | Firewall/IDS/IPS挿入 |

**重要**: Gateway EndpointはPrivateLinkではない。PrivateLinkと呼ばれるのは主にInterface Endpoint / Endpoint Serviceの文脈。

---

## Gateway Endpoint

```text
Private Subnet EC2
  → Route Table
  → S3 Gateway Endpoint
  → Amazon S3
```

### 向いているケース

- VPC内からS3/DynamoDBへプライベートアクセス
- NAT Gateway料金を削減したい
- インターネットゲートウェイを使わずAWSネットワーク内で接続したい

### 制約

- オンプレミス、他リージョン、VPC Peering、Transit Gateway経由からの利用には制約がある
- S3/DynamoDB以外の多くのサービスには使えない

---

## Interface Endpoint

```text
Private Subnet EC2
  → Interface Endpoint ENI
  → AWS service / Endpoint Service
```

### 向いているケース

- Private subnetからKMS / Secrets Manager / ECR / SSM / CloudWatch Logs等へNATなし接続
- オンプレミスからDirect Connect/VPN経由でAWSサービスへプライベート接続
- SaaSや別アカウントサービスにプライベート接続
- セキュリティ要件でインターネット経由を避けたい

---

## PrivateLinkによるサービス公開

サービス提供側はNLBの背後にアプリを置き、Endpoint Serviceとして公開する。利用側はInterface Endpointを作成して接続する。

```text
Provider VPC
  NLB → Service
    ↑
Endpoint Service
    ↑ PrivateLink
Consumer VPC
  Interface Endpoint ENI
```

### 典型シナリオ

- SaaS事業者が顧客VPCへプライベート接続を提供
- 中央アカウントの共有サービスを各アプリVPCから利用
- VPC Peeringを避けてサービス単位で公開
- CIDR重複環境でもサービスアクセスを実現

---

## Endpoint Policy

VPC EndpointにはEndpoint Policyを設定できる。これは「そのEndpoint経由で許可する操作」を制限する追加ガード。

```text
IAM policy / Resource policy / Endpoint policy
  → すべての条件を満たす必要がある
```

例: 特定のS3バケットへのアクセスだけをGateway Endpoint経由で許可する。

---

## PrivateLink vs VPC Peering vs Transit Gateway

| 要件 | 選択肢 |
|---|---|
| VPC間で広く双方向通信 | VPC Peering / Transit Gateway |
| 多数VPCをハブ&スポーク接続 | Transit Gateway |
| 特定サービスだけを公開したい | PrivateLink |
| CIDR重複がある | PrivateLinkが有利 |
| ルート伝播や広いネットワーク到達性が必要 | Transit Gateway |
| S3/DynamoDBへNATなしで安く接続 | Gateway Endpoint |

---

## セキュリティ設計

- Interface EndpointにはSecurity Groupを設定する
- Private DNSを有効にすると通常のサービスDNS名がプライベートIPへ解決される
- Endpoint Policyで操作対象を絞る
- S3 bucket policyで `aws:SourceVpce` を使い、特定Endpoint経由のみ許可できる
- KMSでは `kms:ViaService` と組み合わせるとサービス経由制限を強化できる

---

## よくある誤答パターン

| 誤答 | なぜ危険か |
|---|---|
| S3 Gateway Endpointをオンプレから使う | その用途はInterface Endpointを検討 |
| PrivateLinkでVPC全体を相互接続する | PrivateLinkはサービス単位の接続 |
| Endpoint Policyだけで全権限制御できる | IAM/resource policyも評価される |
| Interface Endpointを作ればNAT Gatewayは全部不要 | インターネット宛や未対応サービスには必要な場合がある |
| VPC PeeringとPrivateLinkを同じものとして扱う | 通信範囲・ルーティング・CIDR重複耐性が異なる |

---

## SAP-C02 Focus

PrivateLinkは、**「VPC同士をつなぐ」ではなく「サービスをプライベートに利用/公開する」** と覚える。S3/DynamoDBならGateway Endpoint、KMS/ECR/SSM/Secrets Manager等ならInterface Endpoint、サービス公開ならEndpoint Service + NLBを選ぶ。

## Official Docs
- Gateway endpoints: https://docs.aws.amazon.com/vpc/latest/privatelink/gateway-endpoints.html
- Interface endpoints: https://docs.aws.amazon.com/vpc/latest/privatelink/create-interface-endpoint.html
- S3 endpoint types: https://docs.aws.amazon.com/AmazonS3/latest/userguide/privatelink-interface-endpoints.html
