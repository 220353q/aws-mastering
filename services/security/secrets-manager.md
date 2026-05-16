# AWS Secrets Manager

## Positioning

AWS Secrets Manager は、データベース認証情報、APIキー、OAuthトークンなどのシークレットを安全に保存・取得・ローテーションするサービス。SAP-C02では、Parameter Storeとの違い、RDS認証情報の自動ローテーション、KMS、クロスリージョンレプリケーションで出題される。

---

## Core Concepts

| 概念 | 内容 |
|---|---|
| Secret | 認証情報やAPIキーなどの機密値 |
| KMS Encryption | 保存時暗号化。CMKを指定可能 |
| Rotation | シークレットと接続先DB/サービスの資格情報を定期更新 |
| Lambda Rotation | カスタムローテーション処理 |
| Managed Rotation | 対応するマネージドシークレットのローテーション |
| Resource Policy | クロスアカウントアクセス制御 |
| Multi-Region Secret | 他リージョンへレプリケーション |

---

## Use Cases

- RDS/AuroraのDB認証情報を自動ローテーションしたい
- アプリケーションコードや環境変数に平文パスワードを置きたくない
- Lambda/ECS/EKSから実行時にシークレットを取得したい
- 複数リージョンアプリで低レイテンシにシークレット取得したい
- 監査可能な形でシークレットアクセスを管理したい

---

## Secrets Manager vs SSM Parameter Store

| 要件 | 選択 |
|---|---|
| DB認証情報の自動ローテーション | Secrets Manager |
| シークレット専用の管理・監査・レプリケーション | Secrets Manager |
| 設定値・階層パラメータ管理 | SSM Parameter Store |
| 低コストで設定値/秘密値を保存 | Parameter Store SecureStringも候補 |
| 複雑なローテーションワークフロー | Secrets Manager |

---

## Rotation Pattern

```text
Application
  → GetSecretValue
  → Secrets Manager
    → KMS decrypt
    → Return current credential

Rotation
  → Secrets Manager schedules rotation
  → Rotation function / managed rotation
  → Update secret + target DB/service credential
```

ローテーションでは、Secrets Manager内の値だけでなく、接続先DBや外部サービス側の資格情報も更新する必要がある。

---

## Cross-Account Access

クロスアカウントでSecretを読む場合は、少なくとも次が必要。

1. Secret側のResource Policy
2. 呼び出し元Principal側のIAM Policy
3. 使用KMSキーのKey Policy / IAM許可

SSE-KMSリソースと同様、**Secretへの権限とKMSへの権限の両方** が必要になる。

---

## SAP-C02 Focus

Secrets Managerを選ぶキーワード:

- automatic rotation
- database credentials
- secret replication across Regions
- cross-account secret access with resource policy
- avoid hard-coded credentials

Parameter Storeを選びやすい文脈:

- アプリ設定値
- 階層パラメータ
- ローテーション要件がない低コストなSecureString

---

## Exam Traps

- Secrets Managerは値を保存するだけではない。ローテーションで接続先資格情報も更新する。
- KMS権限不足だと、SecretのResource Policyが許可していても復号できない。
- クロスリージョンアプリではSecretのレプリケーションを検討する。
- 機密値をLambda環境変数に平文で置く設計は避ける。

---

## Related

- [AWS KMS](kms.md)
- [AWS IAM](iam.md)
- [AWS Systems Manager](../management/ssm.md)

## Official Docs

- https://docs.aws.amazon.com/secretsmanager/latest/userguide/rotating-secrets.html
- https://docs.aws.amazon.com/secretsmanager/latest/userguide/replicate-secrets.html
