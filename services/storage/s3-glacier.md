# Amazon S3 Glacier

Amazon S3 Glacier 系ストレージクラスは、低頻度アクセス/アーカイブ向けのS3ストレージ。SAP-C02では、取り出し時間、コスト、コンプライアンス保持、ライフサイクル移行で問われる。

## 一言で

長期保存と低コストが主目的ならGlacier系。即時アクセスや頻繁な読み取りには向かない。

## ストレージクラスの切り分け

| クラス | 使いどころ |
|---|---|
| S3 Glacier Instant Retrieval | まれにアクセスするがミリ秒取得が必要 |
| S3 Glacier Flexible Retrieval | 数分から数時間の復元でよいアーカイブ |
| S3 Glacier Deep Archive | 最低コストの長期保存。復元は長め |

## 試験で選ぶ条件

- 監査ログや法定保存データを長期保持したい
- 取得頻度は低く、復元時間を許容できる
- S3 LifecycleでStandard/IAから段階的に移行したい
- Object LockやRetentionと組み合わせて改ざん耐性を上げたい

## High-Risk Exam Traps

- Glacier Deep Archiveを即時読み取り用途に選ばない。
- ライフサイクル移行はコスト最適化であり、アクセス要件を先に読む。
- Glacierは別サービスに移すというより、S3ストレージクラスとして扱う。
- 復元後も元オブジェクトのストレージクラスは変わらない。取り出し用コピーの期限に注意する。

## Related

- [S3](s3.md)
- [Storage Options](../../comparisons/storage-options.md)
- [S3 Storage Classes](../../comparisons/s3-storage-classes.md)
