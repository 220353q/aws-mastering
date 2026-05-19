# IAM Boundaries / SCP / Condition Deep Dive

SAP-C02では、IAM Policy、Permission Boundary、SCP、Session Policy、Resource-based Policy、KMS Key Policy、VPC Endpoint Policy、Condition が同じ問題文に混ざる。
混乱の根本は、**権限を付与するもの** と **権限の上限・条件・入口を絞るもの** を同じ「IAM設定」として読んでしまうこと。

このページでは、常に次の4つを分けて読む。

```text
1. 誰に付いているポリシーか
2. 何の最大権限を絞っているか
3. 権限を付与しているのか、上限を作っているだけか
4. ConditionはどのStatementの中で使われているか
```

---

## まず一言で

| 仕組み | 一言でいうと | 権限を付与するか | 主な対象 |
|---|---|---:|---|
| Identity-based Policy | IAM主体に付ける権限本体 | する | IAM User / Group / Role |
| Resource-based Policy | リソース側の入口制御 | する場合がある | S3, KMS, SQS, SNS, Lambdaなど |
| Permission Boundary | IAMユーザー/ロールの最大権限 | しない | IAM User / Role |
| SCP | アカウント/OUの最大権限 | しない | AWS Organizations Account / OU |
| RCP | 組織内リソース側の最大権限 | しない | Organizations配下のリソース |
| Session Policy | 一時セッションの最大権限 | しない | STS / AssumeRole / Federation |
| Condition | Statementを有効にする条件 | ポリシー種別ではない | JSONポリシーのStatement内 |
| VPC Endpoint Policy | Endpoint経由時の追加フィルター | しない | Gateway/Interface Endpoint |
| KMS Key Policy | KMSキー側の入口制御 | する場合がある | KMS Key |
| S3 Block Public Access | S3公開事故を止める強制ブレーキ | しない | S3 Account/Bucket/Access Point |

---

## Permission Boundary と SCP の違い

### Permission Boundary

Permission Boundary は、IAMユーザーまたはIAMロールに設定する **最大権限の境界**。

```text
有効権限 = Identity-based Policy の Allow ∩ Permission Boundary の Allow
```

Permission Boundary自体は権限を付与しない。
たとえば Boundary に `s3:*` がAllowされていても、対象ロールのIdentity-based Policyに `s3:GetObject` がなければS3を読めない。

典型例：

```text
開発者にIAMロール作成を委任したい。
しかし、AdministratorAccess相当のロールを作らせたくない。
→ 作成されるロールにPermission Boundaryを必須化する。
```

### SCP

SCP（Service Control Policy）は、AWS Organizations の Root / OU / Account に適用する **組織・アカウント単位の最大権限の境界**。

```text
有効権限 = IAM等のAllow ∩ SCPで許容された範囲
```

SCPも権限を付与しない。
SCPで `ec2:*` をAllowしても、IAMポリシー側にEC2権限がなければEC2は操作できない。

典型例：

```text
全メンバーアカウントでCloudTrail停止を禁止したい。
特定リージョン以外のリソース作成を禁止したい。
GuardDutyやConfigの無効化を禁止したい。
→ OUまたはAccountにSCPを適用する。
```

### 比較表

| 観点 | Permission Boundary | SCP |
|---|---|---|
| 適用対象 | IAM User / Role | Organizations Root / OU / Account |
| 効く範囲 | 特定IAMエンティティ | アカウント全体、OU全体 |
| 主な目的 | IAM権限の委任を安全にする | 組織全体の統制 |
| 権限付与 | しない | しない |
| 代表パターン | 権限昇格防止 | リージョン制限、セキュリティサービス停止禁止 |
| 覚え方 | 人・ロールの上限 | アカウント・OUの上限 |

---

## 評価ロジックの基本形

試験対策としては、まず次の式で読む。

```text
最終的に許可される条件 =
  明示的Denyがない
  AND SCP/RCPなど組織ガードレールの範囲内
  AND Permission Boundary / Session Policyの範囲内
  AND Identity-based Policy または Resource-based Policy でAllowされている
  AND KMS / VPC Endpoint / S3 Block Public Access など追加レイヤーを満たす
```

最重要ルール：

```text
明示的Denyは、どのAllowよりも優先される。
```

---

## Condition はポリシー種別ではない

ここが最重要。

Condition は、Permission Boundary や SCP のような独立したポリシー種別ではない。
JSONポリシーの `Statement` の中に書ける **条件要素**。

```text
Policy
└─ Statement
   ├─ Effect
   ├─ Principal      ※Resource-based Policyではよく使う
   ├─ Action
   ├─ Resource
   └─ Condition      ← ここ
```

つまり：

```text
Identity-based Policy / Resource-based Policy / SCP / RCP / Permission Boundary / Session Policy
= ポリシーの種類、配置場所、適用レイヤー

Condition
= そのStatementが有効になる条件
```

「IAM Policy Condition」という言い方は広い意味では通じるが、独立ジャンルではない。
正確には **IAM JSON policy の Condition要素** または **ポリシーStatement内のConditionブロック** と理解する。

---

## Resource-based Policy と Condition の関係

Resource-based Policy と Condition は別物だが、対立する概念ではない。
Resource-based Policy の中に Condition を書ける。

例：S3 Bucket Policy は Resource-based Policy。その中にConditionを書ける。

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyOutsideOrg",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::example-bucket/*",
      "Condition": {
        "StringNotEquals": {
          "aws:PrincipalOrgID": "o-xxxxxxxxxx"
        }
      }
    }
  ]
}
```

この場合：

```text
S3 Bucket Policy = Resource-based Policy
Condition        = Resource-based Policy内の条件句
aws:PrincipalOrgID = Conditionで参照する条件キー
```

したがって、S3 Bucket PolicyにConditionを書いたものを「IAM Policy Condition」と呼ぶと少し雑。
正確には **S3 bucket policy の Condition**、または **resource-based policy に含まれる Condition**。

---

## タグ条件と ABAC

タグ条件は Condition の一種。
タグや属性を使って動的に権限を制御する設計思想を ABAC（Attribute-Based Access Control）という。

代表的な条件キー：

| 条件キー | 意味 |
|---|---|
| `aws:PrincipalTag/KeyName` | 実行者に付いたタグを見る |
| `aws:ResourceTag/KeyName` | 対象リソースに付いたタグを見る |
| `aws:RequestTag/KeyName` | 作成・変更リクエストで指定されたタグを見る |
| `aws:TagKeys` | リクエスト内のタグキー一覧を見る |

例：プリンシパルとリソースの `Project` タグが一致するときだけ許可する。

```json
{
  "Effect": "Allow",
  "Action": "ec2:StartInstances",
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "aws:ResourceTag/Project": "${aws:PrincipalTag/Project}"
    }
  }
}
```

注意点：

- すべてのAWSサービス/アクションがすべての条件キーに対応しているわけではない
- サービスごとの対応条件キーは Service Authorization Reference で確認する
- タグの付与・変更権限を制御しないと、タグを書き換えて権限昇格できる
- `aws:RequestTag` と `aws:TagKeys` を使い、許可されたタグだけを作成・更新できるようにする

---

## よく使うConditionキー

| 条件キー | 使いどころ |
|---|---|
| `aws:SourceIp` | 特定IPからのみ許可/拒否 |
| `aws:SourceVpc` | 特定VPCからのみ許可/拒否 |
| `aws:SourceVpce` | 特定VPC Endpoint経由のみ許可/拒否 |
| `aws:PrincipalOrgID` | 自社Organizations配下だけ許可 |
| `aws:MultiFactorAuthPresent` | MFA済みのみ許可 |
| `aws:PrincipalTag/*` | 実行者タグで制御 |
| `aws:ResourceTag/*` | リソースタグで制御 |
| `aws:RequestTag/*` | 作成・変更リクエスト内のタグで制御 |
| `aws:TagKeys` | 使えるタグキーを制限 |
| `aws:RequestedRegion` | 利用可能リージョンを制限 |
| `aws:SecureTransport` | TLS通信のみ許可、非TLSを拒否 |
| `kms:ViaService` | KMSキーを特定AWSサービス経由に限定 |
| `s3:prefix` | S3 ListBucket時のプレフィックス制御 |

---

## Session Policy

Session Policy は、STS AssumeRole やフェデレーションで作成する一時セッションに渡すポリシー。
ロールやフェデレーション元の権限を、一時的にさらに狭める。

```text
有効権限 = ロール/ユーザーの権限 ∩ Session Policy
```

Session Policyも権限を付与しない。
元のロールが持たない権限をSession Policyだけで追加することはできない。

試験キーワード：

- STS AssumeRole
- SAML / OIDC Federation
- 一時認証情報
- セッション単位でS3バケットやプレフィックスを限定
- 共通ロールを使いつつ、利用者ごとに権限を狭める

---

## VPC Endpoint Policy

VPC Endpoint Policy は、VPCエンドポイント経由でAWSサービスにアクセスするときの追加フィルター。

```text
IAMでAllow
かつ Resource-based Policyの条件を満たす
かつ VPC Endpoint Policyで許可される
かつ 明示的Denyがない
→ 許可
```

VPC Endpoint Policyも権限を付与しない。
通信経路に置く「通行許可ゲート」として考える。

よくある組み合わせ：

```text
S3 Gateway Endpoint
+ Endpoint Policyで対象バケットを限定
+ Bucket Policyでaws:SourceVpceを強制
```

---

## KMS Key Policy

KMS Key Policy は、KMSキーを誰がどの条件で使えるかを決めるリソース側ポリシー。

S3オブジェクトの `s3:GetObject` が許可されていても、そのオブジェクトがSSE-KMSで暗号化されており、`kms:Decrypt` が許可されていなければ読めない。

```text
S3 GetObject は許可
でも KMS Decrypt が不可
→ 暗号化オブジェクトを読めない
```

頻出パターン：

- クロスアカウントでS3オブジェクトは見えるが復号できない
- CloudTrailログをSSE-KMSで暗号化したら別アカウントから読めない
- EBSスナップショット共有時にKMSキー利用権限が不足
- Secrets Manager / RDS / S3 / CloudWatch Logs とKMSの組み合わせ

---

## S3 Block Public Access

S3 Block Public Access は、S3のパブリック公開を防ぐ強制ブレーキ。

Bucket Policy や ACL で public access を許可しても、Block Public Access が有効なら公開できないことがある。

```text
Bucket PolicyでPublic Allow
でも Block Public Accessでブロック
→ 公開されない
```

S3公開系では次をまとめて見る。

- Bucket Policy
- Object ACL / Bucket ACL
- Object Ownership
- Block Public Access
- Access Point Policy
- IAM Policy
- SCP / RCP
- KMS Key Policy

---

## RCP：Resource Control Policy

RCP（Resource Control Policy）は、AWS Organizationsでリソース側の最大権限を制御するためのポリシー。

```text
SCP = このアカウントの人・ロールが何をできるかを制限
RCP = この組織内のリソースに対して何を許すかを制限
```

RCPも権限を付与しない。
Identity-based Policy や Resource-based Policy がリソースに対して許可できる最大範囲を制限する。

---

## 「誰を絞っているか」で分類する

```text
人・ロールを絞る
→ Identity-based Policy / Permission Boundary / Session Policy

アカウント・OUを絞る
→ SCP

リソース側で絞る
→ Resource-based Policy / KMS Key Policy / S3 Bucket Policy / RCP

通信経路で絞る
→ VPC Endpoint Policy / aws:SourceVpce / aws:SourceVpc

条件で絞る
→ Condition / ABAC / MFA / IP / Region / SecureTransport
```

---

## 試験での見分け方

### Permission Boundary

- IAMロール作成を開発者に委任したい
- ただし作成されるロールの最大権限を制限したい
- 権限昇格を防ぎたい
- IAM管理権限を一部委任したい

### SCP

- Organizations配下の全アカウントを統制したい
- OU単位で制限したい
- 特定リージョンの利用を禁止したい
- rootユーザーにも制限をかけたい
- セキュリティサービスの停止を組織全体で防ぎたい

### Session Policy

- AssumeRoleした一時セッションだけ権限を狭めたい
- 共通ロールを使いながら、利用者や作業単位でアクセス範囲を変えたい

### Resource-based Policy

- S3、KMS、SQS、SNS、Lambdaなどリソース側でアクセス制御したい
- クロスアカウントアクセスを許可したい
- `Principal` 要素が必要になる

### Condition

- MFA済みのみ許可したい
- 特定IP/VPC/VPC Endpointからのみ許可したい
- 自社Organizations配下だけ許可したい
- タグ一致時のみ許可したい
- TLS通信のみ許可したい
- 特定リージョンの利用を制限したい

---

## 典型的なひっかけ

### 1. Permission BoundaryにAllowを書けば権限が付く

誤り。Boundaryは上限であり、権限の本体ではない。

### 2. SCPにAllowを書けばアカウントに権限が付く

誤り。SCPはOrganizations配下の最大権限を定義するだけ。

### 3. ConditionはResource-based Policyとは別のポリシー種別である

誤り。ConditionはStatementの条件要素。

### 4. S3 Bucket PolicyにConditionを書いたらIAM Policy Conditionになる

表現としては雑。正確にはResource-based Policy内のCondition。

### 5. S3権限があればSSE-KMS暗号化オブジェクトを読める

不十分。KMS Key Policy/IAM Policyで `kms:Decrypt` が必要。

### 6. VPC Endpointを作れば自動的にアクセス制限される

不十分。Endpoint Policy、Resource-based Policyの `aws:SourceVpce`、IAM権限などを組み合わせる。

---

## まとめ

```text
Permission Boundary
= IAMユーザー/ロール単位の最大権限。権限は付与しない。

SCP
= Organizationsのアカウント/OU単位の最大権限。権限は付与しない。

Session Policy
= STS等の一時セッション単位の最大権限。権限は付与しない。

Resource-based Policy
= リソース側に付けるポリシー。Principalを明示して入口を制御する。

Condition
= ポリシー種別ではなく、Statementが有効になる条件。

タグ条件
= ConditionでPrincipalTag/ResourceTag/RequestTag/TagKeys等を使うもの。

ABAC
= タグや属性を使って動的に権限を制御する設計思想。

VPC Endpoint Policy
= 通信経路上の追加フィルター。権限は付与しない。

KMS Key Policy
= 暗号化鍵利用の重要な入口制御。

S3 Block Public Access
= S3公開事故を防ぐ強制ブレーキ。

RCP
= Organizationsでリソース側の最大権限を制御する仕組み。
```

SAP-C02では、サービス名だけでなく、**どのレイヤーで拒否されているか**、**どの制御が権限を付与し、どの制御が上限を作るだけなのか** を読む。

---

## 公式参照

- IAM policy types: https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html
- IAM JSON policy Condition element: https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition.html
- AWS Organizations SCP: https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html
- AWS Organizations RCP: https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_rcps.html
- AWS service authorization reference: https://docs.aws.amazon.com/service-authorization/latest/reference/reference_policies_actions-resources-contextkeys.html
