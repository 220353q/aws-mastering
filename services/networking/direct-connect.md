# AWS Direct Connect — SAP-C02重点ノート

## Overview
AWS Direct Connect は、オンプレミスやコロケーション環境からAWSへ専用ネットワーク接続を確立するサービス。SAP-C02では、**VPNとの違い、Direct Connect Gateway、Transit Gateway連携、冗長化、暗号化要件** が問われる。

Direct Connectは「インターネットを通らない専用接続」だが、**それ自体でIPsec暗号化を提供するものではない**。暗号化が明示要件なら、Site-to-Site VPN over Direct Connect やアプリケーション/TLS/MACsec等を検討する。

---

## Direct Connectを選ぶ要件

| 要件 | 判断 |
|---|---|
| 安定した帯域・低ジッター | Direct Connect |
| インターネットVPNより一貫した性能 | Direct Connect |
| 専用線でオンプレとAWSを接続 | Direct Connect |
| すぐに安く接続したい | Site-to-Site VPN |
| IPsec暗号化が必須 | VPN、またはVPN over DX等 |
| 多数VPC/多数アカウントに集約接続 | DX + TGW / DX Gateway |

---

## VIFの種類

| VIF | 接続先 | 典型用途 |
|---|---|---|
| Private VIF | VPC / VGW / DX Gateway | プライベートIPでVPCへ接続 |
| Public VIF | AWS public services | S3等のパブリックAWSサービスへ専用線経由接続 |
| Transit VIF | Direct Connect Gateway + Transit Gateway | 複数VPC/複数アカウント/大規模接続 |

Endpoint / ENI / VIF の用語横断整理は [AWS Endpoint / ENI / VIF 完全整理](../../comparisons/endpoints-eni-vif.md) を参照。

---

## Direct Connect Gateway

Direct Connect Gateway は、Direct Connect connection と複数リージョンのVPC/TGWを接続するためのグローバルリソース。

```text
On-premises
   │
Direct Connect
   │
Direct Connect Gateway
   ├─ VPC A / VGW
   ├─ VPC B / VGW
   └─ Transit Gateway
```

注意点として、Direct Connect Gateway自体がVPC間通信のトランジットルーターになるわけではない。VPC間通信を集約制御したい場合はTransit Gatewayを使う。

---

## Direct Connect + Transit Gateway

大規模環境では、Transit Gatewayを中心にVPCとオンプレを接続する。

```text
On-premises
  → Direct Connect
  → Direct Connect Gateway
  → Transit Gateway
  → spoke VPCs
```

### 向いているケース

- 数十〜数百VPCへオンプレ接続
- 複数アカウントのネットワーク集約
- ルートテーブルで環境分離
- Direct Connect接続を共有したい

---

## Direct Connect + Site-to-Site VPN

Direct Connectは専用接続だが暗号化は別論点。IPsec暗号化が必要な場合、VPN over DXを検討する。

```text
On-premises
  → Direct Connect Public VIF
  → AWS Site-to-Site VPN endpoint
  → IPsec VPN
  → VPC / Transit Gateway
```

試験では、「専用線の一貫した性能」と「暗号化」の両方が要件にある場合に出やすい。

---

## 冗長化設計

| レベル | 設計 |
|---|---|
| 最小 | 1 connection + VPN backup |
| 標準 | 2 connections、別デバイス/別ロケーション |
| 高可用 | 複数DXロケーション + 複数ルーター + VPN backup |
| ミッションクリティカル | Resiliency Toolkitの高可用モデルに沿う |

Direct Connectは単一接続だと単一障害点になり得る。SAP-C02では「本番」「金融」「高可用性」「メンテナンス影響最小化」が出たら冗長DXを疑う。

---

## Direct Connect vs VPN vs PrivateLink

| サービス | 目的 |
|---|---|
| Site-to-Site VPN | インターネット経由のIPsec接続 |
| Direct Connect | 専用接続でオンプレとAWSを接続 |
| Transit Gateway | 多数VPC/オンプレ接続のハブ |
| PrivateLink | サービス単位でプライベート接続を公開/利用 |
| VPC Peering | 2 VPC間の直接接続 |

---

## よくある誤答パターン

| 誤答 | なぜ危険か |
|---|---|
| Direct Connectだけで通信は暗号化される | IPsec暗号化は別途必要 |
| Direct Connect GatewayでVPC間通信できる | DX GatewayはVPC間トランジット用途ではない |
| 小規模/短期接続でも必ずDX | VPNの方が速く安く導入できる場合がある |
| 1本のDXで高可用性を満たす | 接続/ロケーション/ルーター冗長が必要 |
| PrivateLinkとDXを同じものとして扱う | PrivateLinkはサービス公開/利用、DXは回線接続 |

---

## SAP-C02 Focus

Direct Connectは、**ハイブリッド接続の性能・安定性・冗長性・暗号化要件** の組み合わせで選ぶ。特に、DX + TGW、DX + VPN、DX Gateway、Private VIF / Public VIF / Transit VIFの切り分けを押さえる。

## Official Docs
- AWS Direct Connect connectivity options: https://docs.aws.amazon.com/whitepapers/latest/aws-vpc-connectivity-options/aws-direct-connect.html
- Direct Connect + Transit Gateway: https://docs.aws.amazon.com/whitepapers/latest/aws-vpc-connectivity-options/aws-direct-connect-aws-transit-gateway.html
- Direct Connect + Site-to-Site VPN: https://docs.aws.amazon.com/whitepapers/latest/aws-vpc-connectivity-options/aws-direct-connect-site-to-site-vpn.html

## このページを読んだあとに戻るべき関連ページ

- [AWS Endpoint / ENI / VIF 完全整理](../../comparisons/endpoints-eni-vif.md)
- [Networking Connectivity Options](../../comparisons/networking-options.md)
- [AWS PrivateLink / VPC Endpoints](privatelink.md)
- [Amazon VPC](vpc.md)
