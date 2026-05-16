# AWS Global Accelerator

## Positioning

AWS Global Accelerator は、AWSグローバルネットワークとAnycastの静的IPを使って、グローバルユーザーからリージョナルアプリケーションへの到達性・性能・フェイルオーバーを改善するサービス。

CloudFrontと混同しやすいが、Global Acceleratorは **キャッシュではない**。TCP/UDPアプリケーションや、HTTP以外のグローバル低レイテンシ、固定Anycast IP、リージョン切替で出題される。

---

## Core Concepts

| 概念 | 内容 |
|---|---|
| Accelerator | グローバル入口 |
| Static Anycast IP | ユーザーに見せる固定IP。AWS Edgeで広告される |
| Listener | TCP/UDPとポートを定義 |
| Endpoint Group | リージョン単位のエンドポイント集合 |
| Endpoint | ALB, NLB, EC2, Elastic IP など |
| Traffic Dial | リージョンごとの流量制御 |
| Health Check | 正常なエンドポイントへルーティング |

---

## Static Anycast IPの利点

Static Anycast IP は、ユーザーや取引先に見せる固定のグローバル入口。背後のALB/NLB/EC2/EIPやリージョンを変えても、クライアント側の接続先を安定させやすい。

| 利点 | 何が嬉しいか |
|---|---|
| 許可リスト登録しやすい | 顧客/取引先のFirewallに固定IPを登録できる |
| DNS TTLに依存しにくい | Route 53 failoverよりクライアント/リゾルバキャッシュの影響を受けにくい |
| リージョン追加/移行がしやすい | ユーザー側の接続先IPを変えず、背後のendpoint groupを変えられる |
| 近いEdgeに入れる | ユーザーは近いAWS Edgeへ入り、その後AWS global networkを通る |
| 複数リージョンへ分散できる | endpoint health、weight、traffic dialで誘導できる |

```text
Users / Partners
  → Static Anycast IPs
     → AWS Edge closest to users
        → AWS global network
           → healthy regional endpoint
```

注意:
- 「固定IPがほしいだけ」ならNLBやElastic IPで足りる場合がある。
- **グローバルAnycast + 高速リージョン切替 + TCP/UDP対応** がそろうとGlobal Acceleratorが強い。
- Static IPはacceleratorを削除すると失われる。誤削除防止も運用設計に入れる。

---

## Global Accelerator vs CloudFront

| 要件 | 選択 |
|---|---|
| HTTP/HTTPSコンテンツをキャッシュしたい | CloudFront |
| 静的/動的Webをエッジキャッシュ・TLS終端・OACで配信 | CloudFront |
| TCP/UDPアプリをグローバル高速化 | Global Accelerator |
| 固定Anycast IPが必要 | Global Accelerator |
| リージョン障害時に正常リージョンへ高速切替 | Global Accelerator |
| WAFでWeb L7保護 | CloudFront/ALB/API Gateway + WAF |

---

## Typical Architecture

```text
Global Users
  → AWS Global Accelerator Static Anycast IPs
    → Endpoint Group: ap-northeast-1
       → ALB / NLB / EC2
    → Endpoint Group: us-east-1
       → ALB / NLB / EC2
```

---

## Use Cases

- グローバルユーザーのレイテンシを下げたい
- DNSキャッシュに左右されにくいリージョンフェイルオーバーが必要
- 固定IPを顧客に許可リスト登録してもらう必要がある
- ゲーム、VoIP、金融取引、IoTなどTCP/UDP通信を最適化したい
- ALB/NLB/EC2/EIPを複数リージョンで公開したい

---

## SAP-C02での読み方

Global Acceleratorを選ぶキーワード:

- static anycast IP
- non-HTTP / TCP / UDP
- improve global availability and performance
- fast regional failover without relying only on DNS TTL
- preserve client IP in supported configurations

CloudFrontを選ぶキーワード:

- cache static/dynamic HTTP content
- S3 origin / ALB origin
- OAC / signed URL / edge caching
- WAF at edge

---

## Exam Traps

- Global Acceleratorはキャッシュしない。キャッシュ要件ならCloudFront。
- Route 53 failoverはDNSベース。クライアント/リゾルバのTTL影響を受ける可能性がある。
- 固定IP要件はNLBでも満たせる場合があるが、グローバルAnycastとリージョン切替ならGlobal Accelerator。
- WAF保護はGlobal Accelerator自体ではなく、背後のALBやCloudFront等で考える。

---

## Related

- [Networking Foundations Deep Dive](../../comparisons/networking-foundations-deep-dive.md)
- [Amazon Route 53](route53.md)
- [Amazon CloudFront](cloudfront.md)
- [Elastic Load Balancing](elb.md)
- [Disaster Recovery Pattern](../../patterns/disaster-recovery.md)

## Official Docs

- https://docs.aws.amazon.com/global-accelerator/latest/dg/what-is-global-accelerator.html
- https://docs.aws.amazon.com/global-accelerator/latest/dg/introduction-how-it-works.html

## このページを読んだあとに戻るべき関連ページ

- [Networking Foundations Deep Dive](../../comparisons/networking-foundations-deep-dive.md)
- [Networking Connectivity Options](../../comparisons/networking-options.md)
- [AWS Gateway Services and Terms](../../comparisons/aws-gateways.md)
- [Route 53](route53.md)
- [CloudFront](cloudfront.md)
