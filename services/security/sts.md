# AWS Security Token Service

AWS Security Token Service (AWS STS) は、AssumeRoleなどで一時的な認証情報を発行するサービス。SAP-C02では、クロスアカウント、フェデレーション、Cognito Identity Pool、IAM Roleの裏側として出る。

## 一言で

長期アクセスキーではなく、一時クレデンシャルを発行する基盤がSTS。

## 試験で選ぶ条件

- クロスアカウントでAssumeRoleしたい
- 外部IdP/SAML/OIDCからAWSロールへフェデレーションしたい
- アプリやCI/CDに短期認証情報を使わせたい
- Cognito Identity Pool経由でAWSリソースへ制限付きアクセスしたい

## 重要概念

| 概念 | 意味 |
|---|---|
| AssumeRole | IAM Roleに切り替え、一時認証情報を得る |
| Trust Policy | 誰がそのRoleを引き受けられるか |
| Permission Policy | 引き受けた後に何ができるか |
| External ID | サードパーティ委任時の混同代理問題対策 |

## High-Risk Exam Traps

- Trust Policyで引き受けを許可しても、Permission Policyがなければ操作できない。
- Resource policyやKMS key policyが必要なサービスでは、STSだけでは足りない。
- 長期アクセスキーを配る設計は、ほとんどのSAP-C02問題で弱い。

## Related

- [IAM](iam.md)
- [Cognito](cognito.md)
- [KMS](kms.md)

## SAP-C02での読み方

STSは単独サービスとしてより、AssumeRole、Cognito Identity Pool、SAML/OIDCフェデレーションの裏側として読む。Trust policyは「誰がRoleを引き受けられるか」、Permission policyは「引き受けた後に何ができるか」。KMSやS3のresource policyが必要な場面では、STSだけでは完結しない。

## このページを読んだあとに戻るべき関連ページ

- [Access Control and Encryption Deep Dive](../../comparisons/access-control-and-encryption.md)
- [IAM](iam.md)
- [Cognito](cognito.md)
- [KMS](kms.md)
