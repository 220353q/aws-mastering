# Networking & Content Delivery

## Tier 1

| サービス | 詳細 | 主な用途 |
|---|---|---|
| Amazon VPC | [vpc.md](vpc.md) | サブネット、ルート、NAT、Endpoint、セキュリティグループ |
| Amazon Route 53 | [route53.md](route53.md) | ルーティングポリシー、フェイルオーバー、ヘルスチェック |
| Amazon CloudFront | [cloudfront.md](cloudfront.md) | キャッシュ、OAC、カスタムオリジン、TLS |
| Elastic Load Balancing | [elb.md](elb.md) | ALB / NLB / GWLB |
| AWS Transit Gateway | [transitgateway.md](transitgateway.md) | ハブ&スポーク、マルチアカウント接続 |
| AWS Direct Connect | [direct-connect.md](direct-connect.md) | 専用線、DX Gateway、Transit VIF、VPN over DX |
| AWS Site-to-Site VPN | [site-to-site-vpn.md](site-to-site-vpn.md) | IPsec VPN、DXバックアップ、TGW接続 |
| AWS PrivateLink / VPC Endpoints | [privatelink.md](privatelink.md) | サービス単位のプライベート接続、Gateway/Interface Endpoint |
| AWS Global Accelerator | [global-accelerator.md](global-accelerator.md) | Anycast IP、低レイテンシ、リージョン切替 |

## Design Focus

Multi-AZ、multi-region、private connectivity、ハイブリッド接続、グローバル配信を、可用性・セキュリティ・コスト・運用負荷で比較する。

詳しくは [Networking Options Comparison](../../comparisons/networking-options.md) を参照。
