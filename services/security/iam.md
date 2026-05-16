# AWS IAM & Organizations

## Overview
AWS Identity and Access Management (IAM) は AWS サービス・リソースへのアクセスを制御する中核サービス。AWS Organizations は複数アカウントを OU 単位で統制するガバナンス基盤。SAP-C02 では、単に「IAM を使う」ではなく、**複数ポリシーが同時に存在するときの有効権限の導出**、**クロスアカウントアクセス**、**権限委任**、**Organizations による統制**が問われる。

---

## ポリシー種別と適用範囲

| ポリシー種別 | 適用対象 | 主な用途 | 権限付与するか |
|---|---|---|---|
| **Identity-based Policy** | IAM User / Group / Role | 通常の権限付与 | 付与する |
| **Resource-based Policy** | S3 / KMS / Lambda / SQS など | リソース側から Principal を許可。クロスアカウントで頻出 | 付与する |
| **Permission Boundary** | IAM User / Role | IAM エンティティが持てる最大権限を制限 | 付与しない |
| **SCP (Service Control Policy)** | Organizations の Root / OU / Account | メンバーアカウントの最大権限を制限 | 付与しない |
| **RCP (Resource Control Policy)** | Organizations 配下のリソース | リソース側の最大権限を制限 | 付与しない |
| **Session Policy** | AssumeRole / Federation の一時セッション | 一時クレデンシャルの追加制限 | 付与しない |
| **ACL** | S3 など一部リソース | レガシーなアクセス制御。原則はポリシー優先 | 付与する場合あり |

---

## 権限評価の核心: 「順番」より「集合演算」で考える

IAM 評価は、単純に「SCP を見て、Resource-based Policy を見て、Allow なら終了」という一直線の処理として覚えると危険。SAP-C02 では、S3 bucket policy、KMS key policy、IAM role、Permission Boundary、SCP が同時に登場するため、**有効権限は複数ポリシーの和集合・積集合・明示的 Deny で決まる**と考える。

### 基本式

```
リクエストが許可される条件 =
  明示的 Deny がどこにも存在しない
  AND Organizations のガードレール（SCP/RCP）の範囲内
  AND Permission Boundary / Session Policy の範囲内
  AND Identity-based Policy または Resource-based Policy の Allow が存在する
```

### 評価の優先ルール

| ルール | 意味 | SAP-C02での落とし穴 |
|---|---|---|
| **デフォルトは暗黙 Deny** | Allow がなければ拒否 | 「Deny が書かれていないから許可」は誤り |
| **明示的 Deny は最優先** | どのポリシーの Deny でも Allow を上書き | S3FullAccess があっても bucket policy の Deny で拒否 |
| **Identity-based と Resource-based は基本的に和集合** | どちらかに Allow があれば権限候補になる | Resource-based Policy はクロスアカウント許可で頻出 |
| **Permission Boundary は Identity-based Policy の上限** | Identity Policy と Boundary の積集合 | Boundary だけでは権限は付かない |
| **SCP/RCP は Organizations の上限** | アカウント/リソースの最大権限を制限 | AdministratorAccess でも SCP で禁止された操作は不可 |
| **Session Policy は一時セッションの上限** | AssumeRole 時の追加制限 | ロール自体の権限よりセッションが狭くなる |

### 評価モデル図

```
                          ┌────────────────────┐
                          │      Request        │
                          └─────────┬──────────┘
                                    │
                                    ▼
                     ┌─────────────────────────────┐
                     │  Explicit Deny があるか?      │
                     └─────────┬───────────┬───────┘
                               │Yes        │No
                               ▼           ▼
                            DENY   ┌─────────────────────────────┐
                                   │ Allow の候補があるか?         │
                                   │ Identity-based / Resource-based│
                                   └─────────┬───────────┬───────┘
                                             │No         │Yes
                                             ▼           ▼
                                      DENY(Implicit) ┌──────────────────┐
                                                     │ Boundary/Session │
                                                     │ の範囲内か?       │
                                                     └──────┬─────┬─────┘
                                                            │No   │Yes
                                                            ▼     ▼
                                                          DENY ┌─────────────┐
                                                               │ SCP/RCP の   │
                                                               │ 範囲内か?    │
                                                               └───┬────┬────┘
                                                                   │No  │Yes
                                                                   ▼    ▼
                                                                 DENY ALLOW
```

> 注意: Resource-based Policy の Principal に IAM user ARN、IAM role ARN、role session ARN のどれを指定するかで、Permission Boundary / Session Policy の効き方が変わる場合がある。SAP-C02ではまず「明示的 Deny 最優先」「SCP/Boundary は付与ではなく上限」「Resource-based はリソース側からの許可」という軸で切り分ける。

---

## SCP (Service Control Policy) 詳細

### 特性
- **Organizations の Root / OU / Account に適用**する。
- **Management Account には適用されない**。メンバーアカウント、委任管理者アカウントには適用される。
- SCP は権限を付与しない。**メンバーアカウント内の IAM user / role が実行できる最大権限を制限するガードレール**。
- メンバーアカウントの root user も SCP の制限を受ける。
- OU に適用した SCP は配下 OU / Account に継承される。
- SCP は外部アカウントの Principal に直接は適用されない。たとえば Account A の S3 bucket policy が Account B のユーザーを許可している場合、Account A に付いた SCP は Account B のユーザー自体を制限しない。

### Denyリストモード vs Allowリストモード

| モード | 構成 | メリット | デメリット |
|---|---|---|---|
| **Denyリストモード** | `FullAWSAccess` を残し、禁止したい操作だけ `Deny` | 運用しやすい。既存環境向き | 許可範囲は広め |
| **Allowリストモード** | `FullAWSAccess` を外し、許可サービスだけ `Allow` | 厳格。統制が強い | 新サービス利用時に毎回追加が必要 |

### SCP と IAM Policy の関係

```
有効権限 = IAM で許可された権限 ∩ SCP で許容された権限

例:
  SCP:  S3 と EC2 のみ許容
  IAM:  S3, EC2, RDS を Allow
  結果: S3, EC2 のみ。RDS は SCP の上限外なので不可。
```

### SAP-C02 頻出シナリオ

```
状況: 開発アカウントでは ap-northeast-1 以外の EC2 起動を禁止したい。
誤答: 各開発者の IAM Policy を個別修正する。
正解: 開発 OU に SCP を適用し、Condition: aws:RequestedRegion で制限する。
理由: アカウント数が多い場合は、個別 IAM より Organizations のガードレールが適切。
```

---

## Permission Boundary 詳細

### 用途
Permission Boundary は、IAM ユーザーや IAM ロールが持てる**最大権限**を定義する。Boundary 自体は権限を付与しない。Identity-based Policy が Allow し、かつ Boundary でも Allow されている操作だけが有効になる。

```
有効権限 = Identity-based Policy ∩ Permission Boundary
```

### 権限委任シナリオ

```
状況:
  開発チームに Lambda 用 IAM Role を作らせたい。
  ただし、AdministratorAccess 付き Role を勝手に作られると困る。

設計:
  1. Lambda 実行に必要な範囲だけを Allow した Boundary を作成する。
  2. 開発チームに iam:CreateRole / iam:AttachRolePolicy / iam:PassRole を許可する。
  3. Condition で iam:PermissionsBoundary = 指定Boundary ARN を必須にする。
  4. iam:PassRole も対象 Role / Service Principal を絞る。

結果:
  開発者は Role を作れるが、Boundary 外の権限は付与できない。
```

### Boundary と SCP の違い

| 観点 | Permission Boundary | SCP |
|---|---|---|
| 適用対象 | IAM User / Role | OU / Account |
| 主な目的 | IAM 作成権限の委任制御 | マルチアカウント統制 |
| 権限付与 | しない | しない |
| 試験キーワード | delegate, create roles, prevent privilege escalation | organization, OU, multiple accounts, guardrail |

---

## Resource-based Policy とクロスアカウント

### パターン1: AssumeRole

```
Account B のユーザー/サービス
    → sts:AssumeRole
    → Account A の IAM Role
    → Account A の複数リソースへアクセス
```

**向いているケース**: 複数リソースに横断アクセスする、監査用ロールを作る、運用者が一時的に権限を得る。

### パターン2: Resource-based Policy

```
Account A の S3 bucket policy / KMS key policy / Lambda resource policy
    → Principal に Account B の Role/User を指定
    → Account B から直接アクセス
```

**向いているケース**: 特定リソースだけを共有する。S3 バケット、KMS キー、SQS キューなど。

### S3 + KMS の頻出注意点

S3 オブジェクトを SSE-KMS で暗号化している場合、S3 bucket policy だけでは不十分。アクセス元 Principal には、**S3 の許可**に加えて **KMS key policy または IAM policy 経由の KMS 許可**が必要になる。

---

## IAM Identity Center vs Cognito

| | IAM Identity Center | Amazon Cognito |
|---|---|---|
| 対象ユーザー | 社員・内部ユーザー | アプリのエンドユーザー |
| 主目的 | AWS アカウント / コンソール / CLI へのフェデレーション | Web / Mobile アプリの認証・認可 |
| 代表連携 | Organizations, Permission Set, SAML/OIDC IdP | User Pool, Identity Pool, API Gateway, ALB |
| 試験キーワード | corporate IdP, SSO, multiple AWS accounts | customer login, mobile app, social login |

**鉄則**: 社員が AWS にログインするなら IAM Identity Center。アプリ利用者を認証するなら Cognito。

---

## Use Cases (Tier 1)

1. **Least Privilege Access** — IAM Access Analyzer と CloudTrail で未使用権限を継続的に絞る。
2. **Cross-Account Access** — AssumeRole または Resource-based Policy でアカウント間連携。
3. **Federated Access** — IAM Identity Center + 外部 IdP で社員向け SSO。
4. **Service-to-Service** — EC2 / Lambda / ECS には IAM Role を付与し、長期アクセスキーを避ける。
5. **Governance at Scale** — Organizations + SCP + Control Tower で OU 単位に統制。
6. **Delegated IAM Administration** — Permission Boundary で権限昇格を防ぎながら IAM 作成を委任。

---

## Connections

- **Organizations + Control Tower**: マルチアカウントガバナンス基盤。
- **CloudTrail**: IAM / STS / Organizations API 呼び出しの監査証跡。
- **Access Analyzer**: 外部エンティティへの意図しないリソース共有を検出。
- **GuardDuty**: 異常な IAM / STS 利用を検出。
- **KMS**: Key Policy + IAM Policy + Grants の組み合わせが重要。
- **STS**: AssumeRole による一時クレデンシャル発行。
- **Config**: IAM password policy、root MFA、公開 S3 などの継続評価。

---

## Well-Architected: Security Pillar

| 設計原則 | 実装 |
|---|---|
| 最小権限 | IAM Access Analyzer + CloudTrail による権限レビュー |
| 長期クレデンシャル禁止 | IAM Role + STS。アクセスキーは例外扱い |
| 多層防御 | SCP + Permission Boundary + IAM Policy + Resource Policy |
| 検出と監査 | CloudTrail + GuardDuty + Security Hub |
| 権限委任の安全化 | Permission Boundary + `iam:PermissionsBoundary` 条件 |

---

## SAP-C02 頻出問題パターン

| パターン | キーワード | 正解アプローチ |
|---|---|---|
| 複数アカウント統制 | 100 accounts, OU, compliance | SCP on OU + Control Tower |
| 開発者への権限委任 | developers create roles, prevent privilege escalation | Permission Boundary + Condition |
| 社員向け SSO | corporate IdP, SAML, multiple accounts | IAM Identity Center |
| クロスアカウントアクセス | Account A accesses Account B | AssumeRole or Resource-based Policy |
| S3アクセス拒否 | S3FullAccess but denied | Explicit Deny / bucket policy / SCP / KMS を確認 |
| KMS暗号化S3 | encrypted object, vendor access | S3許可 + KMS key policy/IAM許可 |
| リージョン制限 | only approved Regions | SCP with `aws:RequestedRegion` |

## SAP-C02での読み方

IAM問題は「権限を付与するもの」と「上限を作るもの」を分ける。SCPとPermission BoundaryはAllowを生み出さない。AssumeRoleでは、呼び出し元の`sts:AssumeRole`、引き受け先RoleのTrust policy、RoleのPermission policy、対象リソース側のResource policy/KMS key policyを順に確認する。

## このページを読んだあとに戻るべき関連ページ

- [Access Control and Encryption Deep Dive](../../comparisons/access-control-and-encryption.md)
- [STS](sts.md)
- [KMS](kms.md)
- [Cognito](cognito.md)
- [IAM Identity Center](iam-identity-center.md)
- [Lake Formation](../analytics/lakeformation.md)
