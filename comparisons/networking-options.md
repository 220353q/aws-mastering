# Comparison: AWS Networking Connectivity Options

## Service Selection Table

| Option | Use Case | Latency | Cost | Security / Notes |
|---|---|---|---|---|
| VPC Peering | 少数VPCの直接接続 | Low | Low | 推移的ルーティングなし。大規模では管理複雑 |
| Transit Gateway | 多数VPC/オンプレのハブ&スポーク | Low-Medium | Medium | ルートテーブル分離、マルチアカウント連携 |
| Direct Connect | ハイブリッド高帯域・安定遅延 | Low | High | 専用接続。暗号化は別途VPN/MACsec等を検討 |
| Site-to-Site VPN | 迅速・低コストなハイブリッド接続 | Medium | Low-Medium | IPsec暗号化。インターネット経由 |
| Direct Connect + VPN | 専用接続 + IPsec暗号化 | Low | High | 規制/暗号化要件で頻出 |
| PrivateLink | サービス単位のプライベート公開 | Low | Medium | VPC全体接続ではない。NLB/Endpoint Serviceと関連 |
| Gateway Endpoint | S3/DynamoDBへのプライベートアクセス | Low | Low | S3/DynamoDB専用。NAT削減に有効 |
| Global Accelerator | グローバルAnycast入口・高速フェイルオーバー | Low | Medium-High | キャッシュしない。TCP/UDPにも有効 |
| CloudFront | HTTP(S)配信・キャッシュ・エッジ保護 | Low | Medium | 静的/動的Web、OAC、WAF連携 |

---

## Decision Flow

```text
VPC同士を少数だけ接続？
  → VPC Peering

多数VPC/オンプレを集約？
  → Transit Gateway

オンプレと安定帯域で接続？
  → Direct Connect

暗号化トンネルが必要？
  → Site-to-Site VPN / VPN over Direct Connect

自社サービスを他VPC/他アカウントへプライベート公開？
  → PrivateLink

S3/DynamoDBへのNAT Gatewayコストを削減？
  → Gateway Endpoint

グローバル固定IP・高速リージョン切替？
  → Global Accelerator

HTTPキャッシュ・OAC・エッジ配信？
  → CloudFront
```

---

## Common Exam Traps

- PrivateLinkはVPC間のフルメッシュ接続ではない。サービス単位の接続。
- Direct Connectは専用接続だが、IPsec暗号化が自動で付くわけではない。
- Global Acceleratorはキャッシュしない。キャッシュ要件ならCloudFront。
- VPC Peeringは推移的ルーティング不可。大規模ハブにはTGW。
- Site-to-Site VPNは2本のトンネルを両方設定して冗長化する。

---

## Related

- [Amazon VPC](../services/networking/vpc.md)
- [AWS Transit Gateway](../services/networking/transitgateway.md)
- [AWS Direct Connect](../services/networking/direct-connect.md)
- [AWS Site-to-Site VPN](../services/networking/site-to-site-vpn.md)
- [AWS PrivateLink](../services/networking/privatelink.md)
- [AWS Global Accelerator](../services/networking/global-accelerator.md)
- [Amazon CloudFront](../services/networking/cloudfront.md)
