# AWS CloudTrail

## 何をするサービスか

AWS CloudTrail は、AWSアカウント内のAPI呼び出しやユーザー操作をイベントとして記録する監査サービス。SAP-C02では、**誰が、いつ、どのAPIを、どこから呼んだか**を追跡する用途で出る。

## CloudTrailで見るもの

| イベント種別 | 例 | 試験上の意味 |
|---|---|---|
| Management events | CreateBucket, RunInstances, AttachRolePolicy | デフォルトで重要。管理操作の監査 |
| Data events | S3 Object API, Lambda Invoke | 高頻度。必要な対象に絞って有効化 |
| Insights events | 通常と異なるAPI呼び出し傾向 | 異常な操作パターンの検出 |
| Network activity events | VPC endpoint経由のサービスアクセスなど | ネットワーク経由のアクセス監査 |

## Trail / Event History / CloudTrail Lake

| 機能 | 用途 |
|---|---|
| Event history | 直近の管理イベントを確認する簡易ビュー |
| Trail | 継続的にS3へ証跡を保存。監査・保管向け |
| Organization trail | Organizations配下の全アカウント証跡を集約 |
| CloudTrail Lake | イベントを長期保持しSQLで分析 |

## 典型アーキテクチャ

```text
Member Accounts
  ├─ CloudTrail Organization Trail
  └─ Events
        ↓
Central Audit Account S3 Bucket
        ↓
CloudTrail Lake / Athena / Security Tooling
```

## S3データイベントの注意

S3バケットを作成・削除した操作はManagement eventだが、S3オブジェクトのGetObject/PutObjectなどはData event。大量に発生するため、監査対象バケットだけ有効化する判断が問われる。

## よくある誤答

- **CloudWatch Logsだけで監査する**: CloudWatch Logsはログ基盤。API監査の一次情報はCloudTrail。
- **Configと混同**: Configはリソース設定の履歴。CloudTrailはAPIイベントの履歴。
- **S3オブジェクト操作が自動ですべて記録されると思う**: データイベントは明示的に設定する。

## SAP-C02 Focus

- マルチアカウントではOrganization trailを使う。
- 監査ログは専用ログアーカイブアカウントに集約し、S3 Object LockやKMS、バケットポリシーで保護する。
- API呼び出しをEventBridgeに流し、Systems Manager AutomationやLambdaで自動修復する設計が頻出。
