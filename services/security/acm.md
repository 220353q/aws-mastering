# AWS Certificate Manager - ACM

## Positioning

AWS Certificate Manager は、TLS証明書の発行・管理・更新を支援するサービス。SAP-C02では、CloudFront/ALB/API GatewayでHTTPS化する場面、DNS検証、CloudFront用証明書のリージョン注意、Private CAとの違いが出題される。

---

## Public vs Private Certificates

| 種類 | 用途 | 主な論点 |
|---|---|---|
| Public Certificate | インターネット公開ドメインのTLS | DNS/Email/HTTP検証、ALB/CloudFront/API Gateway連携 |
| Private Certificate | プライベートPKI、社内TLS、mTLS | AWS Private CA、内部サービス間通信 |

---

## Domain Validation

ACMのパブリック証明書では、指定したドメインを所有/制御していることを検証する。

| 方法 | 特徴 |
|---|---|
| DNS validation | 推奨されることが多い。CNAMEで継続的更新しやすい |
| Email validation | ドメイン連絡先メールで検証 |
| HTTP validation | HTTP経由でドメイン制御を検証 |

Route 53を使っている場合、DNS検証レコード作成を簡略化できる。

---

## Regional Considerations

| 利用先 | 証明書のリージョン |
|---|---|
| CloudFront | us-east-1 のACM証明書が必要 |
| ALB | ALBと同じリージョン |
| API Gateway Regional | APIと同じリージョン |
| API Gateway Edge-optimized | us-east-1が関係するケースに注意 |

SAP-C02では、CloudFrontで独自ドメインHTTPSを使う場合の **us-east-1** が罠になりやすい。

---

## ACM with ALB / CloudFront

```text
User
  → HTTPS
  → CloudFront / ALB
    → ACM Certificate
    → Origin / Target Group
```

ACMは証明書管理を簡素化するが、アプリケーション側の認証・認可を代替するものではない。

---

## SAP-C02 Focus

ACMを選ぶ文脈:

- HTTPS/TLS certificate management
- automatic certificate renewal
- custom domain for ALB / CloudFront / API Gateway
- DNS validation with Route 53
- internal private certificates with AWS Private CA

---

## Exam Traps

- CloudFront用のACM証明書はus-east-1で発行する必要がある。
- ACMはTLS証明書管理。ユーザー認証ならCognito/IAM/OIDCなどを考える。
- パブリック証明書ではドメイン所有確認が必要。
- Private CAはコスト・運用責任が増えるため、内部PKI要件がある場合に選ぶ。

---

## Related

- [Amazon CloudFront](../networking/cloudfront.md)
- [Elastic Load Balancing](../networking/elb.md)
- [AWS WAF](waf.md)
- [Amazon Cognito](cognito.md)

## Official Docs

- https://docs.aws.amazon.com/acm/latest/userguide/domain-ownership-validation.html
- https://docs.aws.amazon.com/acm/latest/APIReference/API_RequestCertificate.html
