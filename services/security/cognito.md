# Amazon Cognito — SAP-C02重点ノート

## Overview
Amazon Cognito は、Web / モバイルアプリのユーザー認証・認可を支えるマネージドIDサービス。SAP-C02では、**User Pool と Identity Pool の違い**、API Gateway / ALB / CloudFront / S3 / DynamoDB / IAM Role との連携がよく問われる。

---

## 最重要: User Pool vs Identity Pool

| 項目 | User Pool | Identity Pool |
|---|---|---|
| 主な役割 | ユーザー認証 | AWSリソースへの認可 |
| 扱うもの | ユーザー、パスワード、MFA、IdP連携、JWT | 一時的なAWS認証情報 |
| 連携先 | API Gateway authorizer、ALB認証、アプリログイン | S3、DynamoDB、AppSyncなどAWS API |
| 発行するもの | ID token / access token / refresh token | STS一時クレデンシャル |
| 典型要件 | ログイン、MFA、ソーシャルログイン、SAML/OIDC連携 | ユーザーごとにAWSリソースへ制限付きアクセス |

```text
アプリにログインさせたい → User Pool
ログイン後にAWSリソースへ一時権限を渡したい → Identity Pool
両方必要 → User Poolで認証し、Identity PoolでAWS credentialsに交換
```

---

## User Pool

User Poolはユーザーディレクトリ。サインアップ、サインイン、MFA、パスワードポリシー、属性管理、フェデレーションを提供する。

### よくある連携

```text
Web/Mobile App
  → Cognito User Poolで認証
  → JWTを取得
  → API Gateway / ALB / AppSync へ提示
```

### 試験ポイント

- API GatewayのCognito AuthorizerでJWT検証
- ALBのauthenticate-cognitoでWebアプリ認証
- SAML / OIDC / social IdPとの連携
- MFA、パスワードポリシー、Hosted UI
- Lambda triggersによるサインアップ/認証フローのカスタム

---

## Identity Pool

Identity Poolは、認証済みまたは匿名ユーザーに対して、STSベースの一時的AWS credentialsを発行する。

```text
Cognito User Pool / SAML / OIDC / Social IdP
  → Identity Pool
  → IAM Roleを引き受ける
  → 一時AWS credentials
  → S3 / DynamoDB / AppSync などへアクセス
```

### 試験ポイント

- モバイルアプリからS3へ直接アップロード
- ユーザーごとにDynamoDBの行アクセスを制御
- 認証済みユーザーとゲストユーザーでIAM Roleを分ける
- IAM conditionで `cognito-identity.amazonaws.com:sub` を使う

---

## CognitoとIAM Identity Centerの違い

| 要件 | 選ぶサービス |
|---|---|
| 社員がAWSアカウント/CLI/コンソールへSSOしたい | IAM Identity Center |
| SaaSや社内業務アプリに社員ログインを提供したい | User Poolまたは外部IdP + ALB/API Gateway |
| 一般ユーザー向けWeb/モバイルアプリのログイン | Cognito User Pool |
| アプリユーザーにS3/DynamoDB等の一時AWS権限を渡す | Cognito Identity Pool |

**罠**: Cognitoは「AWS管理者のSSO」ではなく、主にアプリケーションユーザー向け。AWSアカウントへの人間のアクセス管理はIAM Identity Centerが中心。

---

## Poolという言葉に惑わされない

`Pool` は「同じ種類のものをまとめて管理する場所」という意味。CognitoのPoolとRDS ProxyのConnection Poolはまったく別物。

| Pool | 管理するもの | 目的 |
|---|---|---|
| User Pool | アプリユーザー、パスワード、MFA、属性 | ログイン/JWT発行 |
| Identity Pool | 認証済み/匿名IDとIAM Roleの対応 | STS一時AWS credentials発行 |
| Connection Pool | DB接続 | DB接続を再利用して負荷を下げる |

```text
User Pool = あなたは誰?
Identity Pool = AWSで何をしてよい?
Connection Pool = DB接続を使い回す箱
```

詳細は [Pool Terms](../../glossary/pool-terms.md) を参照。

---

## Cognito + API Gateway

```text
Client
  → User Poolでログイン
  → JWT取得
  → API Gateway Cognito Authorizer
  → Lambda / ECS / backend
```

API GatewayでAPIを保護する場合、User Pool tokenを使う。バックエンドがAWSリソースへアクセスする権限は、通常はLambda実行ロールやECS task roleが持つ。

---

## Cognito + S3 直接アップロード

```text
Client
  → User Poolでログイン
  → Identity Poolで一時AWS credentials取得
  → S3へ直接PutObject
```

サーバーを経由せずアップロードしたい場合に使う。ただし、IAM policyをユーザー単位のprefix制限にするなど、最小権限設計が必要。

---

## セキュリティ設計

- MFAを有効化する
- Hosted UI / OIDCフローを適切に選ぶ
- Token有効期限を要件に合わせる
- API側でJWTのaudience / issuer / scopeを検証する
- Identity Poolで渡すIAM Roleを最小権限にする
- CloudTrail / CloudWatch / WAF / Shield との組み合わせを検討する

---

## よくある誤答パターン

| 誤答 | なぜ危険か |
|---|---|
| User PoolだけでS3へ直接アクセスできる | User PoolはJWTを発行するがAWS credentialsは発行しない |
| Identity Poolだけでユーザーディレクトリになる | Identity Poolは認可ブローカーであり認証基盤ではない |
| 社員のAWSコンソールSSOにCognitoを使う | 基本はIAM Identity Center |
| すべてのユーザーに同じIAM Roleを付ける | ユーザー単位/属性単位の制御が崩れる |
| API Gatewayの認証後、バックエンド権限もユーザー権限だと思う | Lambda/ECS実行ロールとエンドユーザー認可は別物 |

---

## SAP-C02での読み方

Cognito問題は、ほぼ必ず **User Pool = 認証 / Identity Pool = AWS一時認証情報** の切り分けで解ける。さらに、IAM Identity Centerとの混同、API Gateway authorizerとの連携、S3直接アップロード時の最小権限が頻出。

## Official Docs
- What is Amazon Cognito?: https://docs.aws.amazon.com/cognito/latest/developerguide/what-is-amazon-cognito.html
- User pools: https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools.html
- Identity pools: https://docs.aws.amazon.com/cognito/latest/developerguide/identity-pools.html

## このページを読んだあとに戻るべき関連ページ

- [Pool Terms](../../glossary/pool-terms.md)
- [Access Control and Encryption Deep Dive](../../comparisons/access-control-and-encryption.md)
- [API Gateway](../integration/apigateway.md)
- [STS](sts.md)
- [IAM Identity Center](iam-identity-center.md)
