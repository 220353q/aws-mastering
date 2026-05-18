# Identity / Policy Evaluation 混同整理 — SAP-C02横断

## 何のためのページか

AWSの権限問題は、単にIAM Policyを読めば終わりではない。SCP、Permission Boundary、Resource Policy、Trust Policy、Session Policy、KMS Key Policy、Endpoint Policyが組み合わさる。

このページでは、SAP-C02で混同しやすいIdentity/Policy系の概念を整理する。

---

## まず結論

```text
IAM Policy = principalが何をできるか
Resource Policy = resourceが誰に許可するか
Trust Policy = 誰がRoleを引き受けられるか
SCP = AWS Organizations上の最大権限ガードレール
Permission Boundary = IAM principal単位の最大権限境界
Session Policy = 一時認証情報セッションの追加制限
KMS Key Policy = KMS keyを誰が使えるかの土台
Endpoint Policy = VPC Endpoint経由で許可する操作の制限
```

---

## 1. IAM Policy vs Resource Policy

| 種類 | 付く場所 | 意味 |
|---|---|---|
| IAM Policy / Identity-based Policy | User / Group / Role | そのprincipalが何をできるか |
| Resource Policy | S3 bucket, KMS key, SQS queue, SNS topic, Lambdaなど | そのresourceに誰がアクセスできるか |

### 例

```text
IAM Role policy:
  このRoleはs3:GetObjectできる

S3 bucket policy:
  このbucketは特定RoleからのGetObjectを許可する
```

クロスアカウントでは、片側だけでは足りないことが多い。

```text
Account A Role側の許可
  +
Account B Resource側の許可
  → クロスアカウントアクセス成立
```

---

## 2. Trust Policy vs Permission Policy

Roleには2種類の重要なpolicyがある。

| 種類 | 何を決めるか |
|---|---|
| Trust Policy | 誰がそのRoleをAssumeRoleできるか |
| Permission Policy | Roleを引き受けた後に何ができるか |

### 覚え方

```text
Trust Policy = 入場許可
Permission Policy = 入場後にできること
```

### 誤答パターン

| 誤答 | なぜ危険か |
|---|---|
| Permission Policyに許可を書けばAssumeRoleできる | Trust Policyで信頼されていないとRoleを引き受けられない |
| Trust Policyにs3:GetObjectを書く | Trust PolicyはRole引き受け元を定義するもの |

---

## 3. SCP vs Permission Boundary

| 種類 | 対象 | 役割 | 権限を付与するか |
|---|---|---|---|
| SCP | AWS OrganizationsのAccount/OU | アカウント内principalの最大権限を制限 | 付与しない |
| Permission Boundary | IAM User/Role | そのprincipalの最大権限を制限 | 付与しない |

### 覚え方

```text
SCP = 組織レベルの天井
Permission Boundary = IAM principal単位の天井
```

どちらも権限を与えない。Allowを有効にするには、Identity-based Policyなどで実際の許可が必要。

---

## 4. Permission Boundary vs Session Policy

| 種類 | いつ効くか | 役割 |
|---|---|---|
| Permission Boundary | IAM principalに設定 | principalの最大権限を定義 |
| Session Policy | AssumeRole等で一時認証情報を発行するとき | そのsessionの権限をさらに絞る |

### 覚え方

```text
Permission Boundary = そのRole/Userの恒常的な境界
Session Policy = 今回発行した一時セッションの追加制限
```

---

## 5. KMS Key Policy vs IAM Policy

KMSは特に混同しやすい。

```text
IAMでkms:Decryptを許可した
  → それだけでは足りない場合がある

KMS Key Policyでkey利用が許可されているか
  → 必ず確認する
```

KMSではKey Policyが土台になる。IAM Policyだけでなく、Key Policy、Grant、サービス経由条件を確認する。

### 典型例

```text
S3 object is encrypted with SSE-KMS
User has s3:GetObject
But kms:Decrypt or key policy is missing
  → 読めない
```

---

## 6. Endpoint Policy vs IAM / Resource Policy

Endpoint Policyは、VPC Endpoint経由で許可する操作を制限する追加ガード。

```text
IAM Policy
Resource Policy
Endpoint Policy
KMS Key Policy
  → 全体で評価される
```

Endpoint Policyだけで最終権限が決まるわけではない。

### 例

```text
S3 bucket policy:
  aws:SourceVpce = vpce-xxxx のときだけ許可

Endpoint Policy:
  特定bucketへのGetObjectだけ許可
```

---

## 7. Explicit Deny

明示的Denyは強い。

```text
Explicit Deny > Allow
```

どこかで明示的Denyがあれば、他でAllowされていても拒否される。

例。

```text
IAM Policy allows s3:DeleteObject
SCP denies s3:DeleteObject
  → Deny
```

---

## 8. 権限評価のざっくりモデル

単純な順番評価ではなく、集合演算で読む。

```text
有効権限 =
  明示的Denyがない
  ∧ SCP/RCPの範囲内
  ∧ Permission Boundary/Session Policyの範囲内
  ∧ Identity-based PolicyまたはResource-based PolicyでAllowされる
```

KMSが絡む場合はさらにKey Policy / Grantも見る。

---

## 9. SAP-C02判断フロー

```text
アクセスできない？
  → 明示的Denyはある？
  → SCPで上限を超えていない？
  → Permission Boundaryで制限されていない？
  → Session Policyで絞られていない？
  → IAM PolicyでAllowされている？
  → Resource Policyで相手側が許可している？
  → KMSならKey Policy/Grantは許可している？
  → VPC EndpointならEndpoint Policyやaws:SourceVpce条件は？
```

---

## 10. よくある誤答パターン

| 誤答 | なぜ危険か |
|---|---|
| SCPでAllowすれば権限が付与される | SCPは権限を付与しない。最大権限を制限する |
| Permission BoundaryでAllowすれば実行できる | Boundaryも権限を付与しない。Identity policyのAllowが必要 |
| IAM PolicyだけでクロスアカウントS3アクセスできる | Resource側のbucket policyも必要になり得る |
| RoleのPermission Policyだけ見てAssumeRole可否を判断する | Trust Policyを見ないといけない |
| S3権限だけ見てSSE-KMS objectを読めると判断する | KMS Decrypt / Key Policyが必要 |
| Endpoint Policyだけでアクセス制御が完結する | IAM/resource/KMS/SCPも評価される |

---

## 最短暗記

```text
SCP = 組織の天井
Permission Boundary = principalの天井
Session Policy = 一時セッションの追加制限
Trust Policy = Roleに入れる人
Permission Policy = Roleに入った後にできること
Resource Policy = resource側の許可
KMS Key Policy = key利用の土台
Endpoint Policy = endpoint経由の制限
Explicit Deny = 最優先
```

---

## 関連ページ

- [AWS 混同しやすい概念インデックス](aws-confusing-concepts-index.md)
- [Endpoint / ENI / VIF / PrivateLink 理解補完レジュメ](endpoints-eni-vif-remedial-resume.md)
- [AWS Endpoint / ENI / VIF 完全整理](endpoints-eni-vif.md)
- [AWS PrivateLink / VPC Endpoints](../services/networking/privatelink.md)
- [KMS](../services/security/kms.md)
