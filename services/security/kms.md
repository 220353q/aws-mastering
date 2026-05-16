# AWS KMS — SAP-C02重点ノート

## Overview
AWS Key Management Service (AWS KMS) は、暗号鍵の作成・保管・利用権限・監査を管理するマネージドサービス。SAP-C02では単独サービスとしてではなく、**S3 / EBS / RDS / Aurora / DynamoDB / CloudTrail / Secrets Manager / DMS / クロスアカウントアクセス** と組み合わせて問われる。

KMSの本質は「暗号化するサービス」ではなく、**鍵を誰が、どのサービス経由で、どの条件で使えるかを制御するサービス** と捉える。

---

## 試験で問われる核心

### 1. KMS key policy が最上位の入口
KMS key には必ず key policy が存在する。IAM policy だけで KMS key を使えるわけではない。key policy が IAM による権限委任を許可している場合に、IAM policy と組み合わせてアクセスを許可できる。

```text
KMS key を使えるか？
  = key policy が入口を開いている
  + IAM policy / grant / service integration が適切
  + 明示的Denyがない
  + 条件キーが一致する
```

**SAP-C02の罠**: IAM role に `kms:Decrypt` を付けても、KMS key policy 側でその principal またはアカウント利用が許可されていなければ失敗する。

---

### 2. Envelope Encryption
多くのAWSサービスは、KMS keyでデータ本体を直接暗号化するのではなく、データキーを使う。

```text
[Customer Data]
   ↓ Data key で暗号化
[Encrypted Data]

[Data key]
   ↓ KMS key で暗号化
[Encrypted Data Key]
```

読み取り時は、KMS keyでEncrypted Data Keyを復号し、得られたData Keyで実データを復号する。

---

### 3. AWS managed key vs Customer managed key

| 種別 | 管理主体 | Key policy編集 | ローテーション | SAP-C02での判断 |
|---|---|---:|---|---|
| AWS owned key | AWS | 不可 | AWS管理 | 利用者からはほぼ見えない |
| AWS managed key | AWS | 不可 | AWS管理 | 簡単だが細かな制御不可 |
| Customer managed key | 顧客 | 可能 | 設定可能 | クロスアカウント・監査・条件制御で重要 |

試験では、**要件に「監査」「キー管理」「クロスアカウント」「特定条件でのみ復号」「削除・ローテーション制御」** が出ると Customer managed key が選ばれやすい。

---

## 権限モデル

### Key policy
KMS key のリソースポリシー。誰がそのkeyを管理・使用できるかを決める。

### IAM policy
principal 側の権限。key policy が IAM の使用を許可している場合に有効。

### Grants
一時的またはサービス連携のために、KMS keyの使用権限を委譲する仕組み。EBS、RDS、DMSなど、AWSサービスが内部的に利用するケースがある。

```text
Key policy: 鍵側の入口
IAM policy: 利用者側の権限
Grant: 一時的・委譲的な利用許可
Condition: どの経路・文脈で使えるかの制約
```

---

## 暗号化は誰から何を隠すのか

| 仕組み | 守る場所 | 主な目的 |
|---|---|---|
| TLS / HTTPS | 通信経路 | ネットワーク上の盗聴・改ざんを防ぐ |
| SSE-S3 | S3保存時 | S3管理鍵で保存データを暗号化 |
| SSE-KMS | S3保存時 + KMS制御 | KMS keyで鍵利用を監査/制御する |
| EBS/RDS encryption | ブロック/DB保存時 | スナップショット含む保存データ保護 |
| Lake Formation | データレイク権限 | 行/列/テーブル単位で見えるデータを制御 |

**復号** は「暗号化されたデータを読める形に戻すこと」。問題文で「誰が復号できるか」と出たら、KMS key policy、IAM policy、grants、サービス経由条件を確認する。

**重要**: 保存時暗号化はデータ認可の代わりではない。KMSで暗号化しても、Lake Formationの行/列レベル権限やIAM/S3 bucket policyの設計は別に必要。

---

## クロスアカウントKMS

別アカウントのIAM roleがKMS keyを使うには、通常は両側の許可が必要。

```text
Account A: KMS key owner
  - key policyでAccount BのroleまたはAccount B rootを許可

Account B: key user
  - IAM policyで対象KMS keyへの kms:Decrypt 等を許可
```

**頻出パターン**:

- S3バケットはクロスアカウントで読める
- しかしSSE-KMSで暗号化されている
- S3権限はあるのに `AccessDenied` になる
- 原因はKMS key policy / IAM policy不足

---

## `kms:ViaService` とサービス経由制限

KMS key を直接使わせず、S3やEBSなど特定サービス経由でのみ利用させたい場合に条件キーを使う。

```json
{
  "Condition": {
    "StringEquals": {
      "kms:ViaService": "s3.ap-northeast-1.amazonaws.com"
    }
  }
}
```

**試験での意味**: 「S3オブジェクト暗号化には使わせたいが、ユーザーが直接Decryptするのは避けたい」という要件で有効。

---

## Multi-Region keys

Multi-Region keys は、同じ key ID と key material を複数リージョンに複製できるKMS key。クライアント側暗号化、グローバルアプリケーション、DRでの復号継続性に関係する。

ただし、**Multi-Region keyはデータそのものをレプリケートしない**。S3 CRR、DynamoDB Global Tables、Aurora Global Databaseなど、データ側のレプリケーションとは別に考える。

---

## サービス別の出題パターン

### S3 + SSE-KMS
- S3 bucket policyで許可してもKMS権限がないと読めない
- クロスアカウント共有では bucket policy と key policy の両方が必要
- CloudTrailでKMS API利用を監査できる

### EBS / RDS / Aurora
- スナップショット共有時にKMS keyも共有が必要
- AWS managed keyで暗号化されたスナップショットはクロスアカウント共有に制約がある
- 移行・DR要件ではCustomer managed keyの方が設計しやすい

### CloudTrail
- CloudTrailログをSSE-KMSで暗号化する場合、CloudTrailサービスがKMS keyを使える必要がある
- 証跡の完全性・監査要件とセットで問われる

### DMS / Migration
- 暗号化されたソース/ターゲット/ログ/中間ストレージを扱う場合、DMSのサービスロールやエンドポイントにKMS権限が必要

---

## よくある誤答パターン

| 誤答 | なぜ危険か |
|---|---|
| IAM policyに `kms:*` を付ければよい | key policyが入口を開いていなければ使えない |
| S3 bucket policyだけでクロスアカウント共有できる | SSE-KMSならKMS側の許可も必要 |
| AWS managed keyで十分 | クロスアカウント・詳細条件・削除制御が必要な場合に不足 |
| Multi-Region keyでデータも自動複製される | key materialの複製であり、データ複製ではない |
| KMS keyを直接アプリに使わせる | サービス経由制限や最小権限を検討すべき |

---

## 意思決定フロー

```text
暗号化が必要？
  └─ AWSサービス既定の暗号化で十分？
       ├─ Yes → AWS managed key / default encryption
       └─ No
          └─ 鍵ポリシー、クロスアカウント、監査、条件制御が必要？
              ├─ Yes → Customer managed key
              └─ No → AWS managed key

クロスアカウントアクセス？
  └─ S3/EBS/RDSなどのリソース権限 + KMS key policy + 利用者IAM policy を確認
```

---

## SAP-C02での読み方

KMSは「暗号化サービス」ではなく、**データアクセス制御の最後の門番** として出題される。S3、RDS、EBS、CloudTrail、DMS、Secrets Manager、クロスアカウント共有の問題では、KMS権限の見落としが典型的な落とし穴になる。

## Official Docs
- AWS KMS key policies: https://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html
- IAM policies with AWS KMS: https://docs.aws.amazon.com/kms/latest/developerguide/iam-policies.html
- Grants in AWS KMS: https://docs.aws.amazon.com/kms/latest/developerguide/grants.html

## このページを読んだあとに戻るべき関連ページ

- [Access Control and Encryption Deep Dive](../../comparisons/access-control-and-encryption.md)
- [IAM](iam.md)
- [S3](../storage/s3.md)
- [RDS / Aurora Connection Deep Dive](../../comparisons/rds-aurora-connection-deep-dive.md)
- [Lake Formation](../analytics/lakeformation.md)
