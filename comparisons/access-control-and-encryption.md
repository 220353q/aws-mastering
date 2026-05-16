# Access Control and Encryption Deep Dive

SAP-C02では、IAM、SCP、AssumeRole、KMS、SSE-KMS、TLS、Lake Formation、QuickSightが同じ問題文に混ざる。混乱の根本は、**認証、認可、暗号化、復号、保存時暗号化、通信経路暗号化、データ表示制御**を同じ「セキュリティ」として読んでしまうこと。

このページでは、常に「誰が」「何に」「どの権限で」「どの経路で」アクセスしているのかを明示する。

## まず一言で

| 概念 | 一言でいうと | AWSでの代表 | これは何ではないか |
|---|---|---|---|
| 認証 | あなたは誰かを確認する | Cognito User Pool, IAM Identity Center, SAML/OIDC | 何をしてよいかの最終判断ではない |
| 認可 | 何をしてよいかを決める | IAM policy, resource policy, Lake Formation | パスワード確認ではない |
| 暗号化 | 読めない形に変換する | SSE-KMS, EBS/RDS encryption, TLS | データ表示制御ではない |
| 復号 | 読める形に戻す | `kms:Decrypt`, service decrypt | S3のRead権限だけではない |
| 保存時暗号化 | 置いてあるデータを守る | S3 SSE, RDS/EBS encryption | 通信中の盗聴対策ではない |
| 通信経路暗号化 | 通っているデータを守る | TLS/HTTPS, VPN, mTLS | 保存データの暗号化ではない |
| データ表示制御 | 行/列/画面を見せ分ける | Lake Formation, QuickSight RLS/CLS | KMS暗号化の代替ではない |

## 流れで見る

```text
User / Service
  1. 認証: 誰か?
     → Cognito / IAM Identity Center / IAM principal / external IdP

  2. 認可: AWS APIやデータ操作をしてよいか?
     → IAM policy / SCP / Permission Boundary / Resource Policy / Lake Formation

  3. 経路: どこを通るか?
     → Internet / VPC endpoint / PrivateLink / VPN / Direct Connect

  4. 通信経路暗号化: 途中で読まれないか?
     → TLS / HTTPS / VPN / mTLS

  5. 保存時暗号化: 保管媒体から読まれないか?
     → SSE-S3 / SSE-KMS / EBS / RDS encryption

  6. 復号: 暗号化データを読める形に戻せるか?
     → KMS key policy + IAM + grants + service context

  7. 表示制御: どの行/列/ダッシュボードを見せるか?
     → Lake Formation / QuickSight / application logic
```

## 認証と認可

### 認証

- 一言でいうと: 「この人/サービスは誰か」を確認する。
- 何のためにあるか: なりすましを防ぐため。
- 何が嬉しいか: ユーザー、社員、サービス、外部IdPを識別できる。
- 何と混同しやすいか: 認可。ログイン成功は「何でもできる」ではない。
- 試験問題ではどう出るか: `login`, `MFA`, `SAML`, `OIDC`, `JWT`, `corporate IdP`。
- 間違えやすい選択肢: MFA済みならS3/KMS/Lake Formation権限も自動でOKと考える。
- 小さな構成図:

```text
User
  → sign in with IdP
  → token / session
  → API request
```

- 暗記のコツ、語源、語呂: **Authentication = AuthN = Name確認**。

### 認可

- 一言でいうと: 「何をしてよいか」を決める。
- 何のためにあるか: 最小権限を実現し、不要な操作を止めるため。
- 何が嬉しいか: `s3:GetObject`は許可、`s3:DeleteBucket`は禁止、のように行動を制御できる。
- 何と混同しやすいか: 暗号化。暗号化しても権限設計は必要。
- 試験問題ではどう出るか: `least privilege`, `cross-account access`, `explicit deny`, `permission boundary`, `resource policy`。
- 間違えやすい選択肢: SCPやPermission Boundaryを「権限付与」として使う。
- 小さな構成図:

```text
Principal
  → calls AWS API
  → IAM evaluation
  → Allow or Deny
```

- 暗記のコツ、語源、語呂: **Authorization = AuthZ = Zが最後、最後に許可判断**。

## IAM / Organizations / STS

### 権限評価の基本式

```text
許可される条件 =
  明示的Denyがない
  AND SCP/RCPなど組織ガードレールの範囲内
  AND Permission Boundary / Session Policyの範囲内
  AND Identity-based policy または Resource-based policy のAllowがある
```

| 仕組み | 何のためにあるか | 何が嬉しいか | 試験の罠 |
|---|---|---|---|
| OU | アカウントをまとめる箱 | 部門/環境ごとに統制できる | OU自体は権限ではない |
| SCP | アカウントの最大権限 | 多数アカウントへガードレール | 権限を付与しない |
| Permission Boundary | IAM User/Roleの最大権限 | IAM作成権限を安全に委任 | Boundaryだけでは許可にならない |
| Identity policy | Principal側の許可 | 通常のAWS API権限 | 明示Denyに負ける |
| Resource policy | Resource側の許可 | S3/SQS/Lambda/KMSの共有 | Principal指定とKMS権限を見落とす |
| Session policy | 一時セッションの追加上限 | AssumeRole時にさらに狭める | Role権限より広げられない |

### AssumeRole / STS

- 一言でいうと: STSで別のIAM Roleとして一時的に振る舞う。
- 何のためにあるか: 長期アクセスキーを配らず、短期権限でクロスアカウントやフェデレーションを実現するため。
- 何が嬉しいか: 期限付き、監査可能、権限をRoleに集約できる。
- 何と混同しやすいか: Resource policy。AssumeRoleは「別Roleに変身」、Resource policyは「リソース側が直接Principalを許可」。
- 試験問題ではどう出るか: `cross-account`, `temporary credentials`, `third-party access`, `external ID`。
- 間違えやすい選択肢: Trust policyだけでS3やKMS操作までできると思う。
- 小さな構成図:

```text
Account B principal
  → sts:AssumeRole
  → Account A role
  → temporary credentials
  → Access resources as Account A role
```

- 暗記のコツ、語源、語呂: **Assume = 役割を引き受ける**。Trustで入場、Permissionで行動。

必要なもの:

| 場所 | 必要な許可 |
|---|---|
| 呼び出し元 | `sts:AssumeRole` |
| 引き受け先Role | Trust policyで呼び出し元を信頼 |
| 引き受け後 | Role permission policy |
| 対象リソース | 必要に応じてresource policy / KMS key policy |

## KMS / SSE-KMS / TLS

### 保存時暗号化と通信経路暗号化

| 観点 | 保存時暗号化 | 通信経路暗号化 |
|---|---|---|
| 守る場所 | ディスク、S3、DB、Snapshot | ネットワーク上 |
| 代表 | SSE-KMS, EBS encryption, RDS encryption | TLS/HTTPS, VPN, mTLS |
| 防ぐもの | 保管媒体からの読み取り | 盗聴、改ざん |
| 罠 | 復号権限が別に必要 | 保存データは別に暗号化が必要 |

### KMS Key Policy

- 一言でいうと: KMS key側の入口。
- 何のためにあるか: 誰が鍵を管理/使用できるかを鍵側で制御するため。
- 何が嬉しいか: IAMだけではなく、鍵そのものに最小権限と監査境界を持てる。
- 何と混同しやすいか: IAM policy。IAMに`kms:Decrypt`があってもkey policyが入口を閉じていたら使えない。
- 試験問題ではどう出るか: `SSE-KMS encrypted object AccessDenied`, `cross-account KMS`, `key policy`。
- 間違えやすい選択肢: S3 bucket policyだけで暗号化オブジェクトを読めると考える。
- 小さな構成図:

```text
Principal
  → s3:GetObject allowed?
  → object encrypted with SSE-KMS?
  → kms:Decrypt allowed by key policy/IAM/grant?
  → data returned
```

- 暗記のコツ、語源、語呂: **KMSは鍵部屋の受付**。S3の入館証だけでは鍵部屋に入れない。

### SSE-KMSの復号フロー

```text
PutObject
  → S3 asks KMS: GenerateDataKey
  → S3 encrypts object with data key
  → S3 stores encrypted object + encrypted data key

GetObject
  → caller must be allowed by S3
  → S3 asks KMS: Decrypt encrypted data key
  → S3 decrypts object
  → caller receives plaintext over TLS if HTTPS
```

ここで「誰が復号しているか」は文脈に注意する。多くのSSE-KMSでは、アプリがKMSから平文データキーを直接受け取るのではなく、S3などのサービスがKMSを呼び、許可されたリクエストに平文データを返す。

### `kms:ViaService`

特定サービス経由でだけKMS keyを使わせたいときに使う条件。

```json
{
  "Condition": {
    "StringEquals": {
      "kms:ViaService": "s3.ap-northeast-1.amazonaws.com"
    }
  }
}
```

これは「S3のSSE-KMS用途では使わせるが、PrincipalがKMS Decrypt APIを直接叩く使い方は避けたい」という設計で効く。

## Lake Formation / Athena / Glue / QuickSight / S3 / KMS

データレイク問題では、権限が複数レイヤーに分かれる。

```text
Analyst / BI user
  → QuickSight dashboard or Athena query
  → execution role / workgroup / data source
  → Glue Data Catalog metadata
  → Lake Formation table/column/row permissions
  → S3 data location
  → KMS decrypt if encrypted with SSE-KMS
```

| レイヤー | 何を見るか | 罠 |
|---|---|---|
| QuickSight | 誰にdashboard/datasetを見せるか | QuickSightだけでS3/KMS権限が完結しない |
| Athena | 誰がクエリを実行し、結果をどこへ書くか | 結果出力先S3/KMS権限を忘れる |
| Glue Data Catalog | テーブル定義/メタデータ | メタデータが見えてもデータが読めるとは限らない |
| Lake Formation | DB/table/column/row/data location権限 | IAMの広いS3権限で迂回させない |
| S3 | オブジェクト保存場所 | bucket policy/IAMとLFの関係を見る |
| KMS | 暗号化データ/結果/メタデータの復号 | `kms:Decrypt`不足でAccessDenied |

### Lake Formation

- 一言でいうと: データレイクの表/行/列レベルの認可レイヤー。
- 何のためにあるか: S3上のデータを、ビジネス単位のテーブル権限として管理するため。
- 何が嬉しいか: IAM/S3 policyだけでは難しい細粒度制御を一元化できる。
- 何と混同しやすいか: Glue、QuickSight。Glueはカタログ/ETL、QuickSightはBI表示。
- 試験問題ではどう出るか: `column-level permissions`, `governed data lake`, `centralized permissions`, `Athena access`。
- 間違えやすい選択肢: KMSで暗号化すれば列レベル制御が不要と考える。
- 小さな構成図:

```text
Athena
  → Lake Formation checks table/column permission
  → grants scoped access to S3 location
  → query returns allowed columns/rows
```

- 暗記のコツ、語源、語呂: **Lake Formationは湖の入館管理**。水そのものはS3、地図はGlue。

### QuickSight

- 一言でいうと: BIダッシュボードと埋め込み分析の表示レイヤー。
- 何のためにあるか: データを業務ユーザーに見せるため。
- 何が嬉しいか: SPICE、Direct Query、RLS/CLS、埋め込みBIをマネージドで使える。
- 何と混同しやすいか: ETL/DWH/データレイク権限。QuickSightは表示であり、Glue/Redshift/Lake Formationの代替ではない。
- 試験問題ではどう出るか: `dashboard`, `embedded analytics`, `row-level security`, `SPICE`。
- 間違えやすい選択肢: QuickSightをETLやDWHとして選ぶ。
- 小さな構成図:

```text
Business user
  → QuickSight dashboard
  → Dataset: SPICE or Direct Query
  → Athena / Redshift / RDS / S3
```

- 暗記のコツ、語源、語呂: **QuickSight = すばやく見る画面**。

## 問題文からの変換表

| 問題文の制約語 | 読み替え | 選びやすい候補 |
|---|---|---|
| corporate users access many AWS accounts | 社員SSO | IAM Identity Center |
| customer login / social login / JWT | アプリユーザー認証 | Cognito User Pool |
| mobile app uploads directly to S3 | AWS一時認証情報 | Cognito Identity Pool |
| developers can create roles but not admin roles | 権限委任の上限 | Permission Boundary |
| all accounts in OU must deny regions | 組織ガードレール | SCP |
| cross-account operational access | 一時的にRoleを引き受ける | AssumeRole / STS |
| allow another account to one S3 bucket | リソース側許可 | Bucket policy + IAM |
| encrypted S3 object access denied | 復号権限不足 | KMS key policy + IAM |
| data lake column-level permissions | データ表示/参照認可 | Lake Formation |
| dashboard row-level filtering | BI表示制御 | QuickSight RLS |
| protect traffic in transit | 通信経路暗号化 | TLS / HTTPS / VPN |
| protect snapshots or disks | 保存時暗号化 | KMS-backed encryption |

## Official Docs

- AWS IAM permissions boundaries: https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html
- AWS KMS key policies: https://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html
- Lake Formation and Athena permissions: https://docs.aws.amazon.com/athena/latest/ug/lf-athena-user-permissions.html
- API Gateway JWT authorizers: https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-jwt-authorizer.html

## SAP-C02での読み方

1. 「ログインできる」と「データを読める」を分ける。
2. 「S3を読める」と「SSE-KMSを復号できる」を分ける。
3. 「暗号化されている」と「行/列を見せ分けられる」を分ける。
4. SCP/Permission Boundaryは権限付与ではなく上限として読む。
5. データレイクでは、QuickSight/Athena/Glue/Lake Formation/S3/KMSのどの層で拒否されるかを追う。
6. 設問のたびに、誰が、何に、どの権限で、どの経路でアクセスしているかをメモする。

## このページを読んだあとに戻るべき関連ページ

- [IAM](../services/security/iam.md)
- [STS](../services/security/sts.md)
- [KMS](../services/security/kms.md)
- [Cognito](../services/security/cognito.md)
- [Lake Formation](../services/analytics/lakeformation.md)
- [QuickSight](../services/analytics/quicksight.md)
- [Analytics / Data Lake Service Selection](analytics-data-lake.md)
- [Networking Foundations Deep Dive](networking-foundations-deep-dive.md)
