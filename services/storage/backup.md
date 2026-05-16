# AWS Backup

## Positioning

AWS Backup は、複数AWSサービスのバックアップを **一元管理・自動化・監査** するサービス。SAP-C02では、単一サービスのスナップショット機能ではなく、**Organizations横断・バックアップポリシー・クロスリージョン/クロスアカウント・コンプライアンス** の文脈で出る。

---

## Core Concepts

| 概念 | 内容 |
|---|---|
| Backup Plan | いつ、どの頻度で、どのライフサイクルでバックアップするか |
| Backup Vault | バックアップを保存する論理コンテナ |
| Backup Policy | Organizations配下のアカウントへポリシー適用 |
| Cross-Region Copy | DR/コンプライアンスのため別リージョンへコピー |
| Cross-Account Copy | ランサムウェア/誤削除対策として別アカウントへコピー |
| Vault Lock | WORM的な保持制御、削除/変更耐性 |
| Audit Manager | バックアップ準拠状況の監査 |

---

## Use Cases

- 複数アカウント・複数リージョンのバックアップを統制したい
- RDS、EBS、EFS、DynamoDB、S3などを横断して保護したい
- 本番アカウントとは別アカウントにバックアップをコピーしたい
- DR要件でバックアップを別リージョンに保持したい
- 監査証跡と保持ポリシーが必要
- ランサムウェア対策として削除耐性を高めたい

---

## Backup vs Native Snapshot

| 要件 | 選択 |
|---|---|
| 1つのRDS/EBSだけを手動で保護 | 各サービスのSnapshot機能でも可 |
| 複数サービスを一元管理 | AWS Backup |
| Organizations配下へ標準ポリシー適用 | AWS Backup + Backup Policy |
| 別アカウント/別リージョン保管 | AWS Backup Cross-Account / Cross-Region Copy |
| コンプライアンス監査 | AWS Backup Audit Manager |

---

## Cross-Region / Cross-Account Design

```text
Production Account / ap-northeast-1
  └─ Backup Vault
       └─ Copy to DR Region
       └─ Copy to Backup Account

Backup Account / Separate OU
  └─ Protected Backup Vault
       └─ Vault Lock / least privilege / monitoring
```

### Why separate account?

- 本番アカウント侵害時にバックアップまで削除されるリスクを下げる
- 運用権限を分離できる
- コンプライアンス監査を集約しやすい

---

## SAP-C02 Focus

AWS Backupを選ぶ文脈:

- 「複数アカウントにまたがるバックアップを一元化」
- 「Organizationsで標準バックアップポリシーを強制」
- 「クロスリージョン/クロスアカウントのバックアップコピー」
- 「監査要件・保持要件・WORM」
- 「運用負荷を下げるマネージドバックアップ」

---

## Exam Traps

- AWS BackupはDR実行環境そのものではない。復旧先のネットワーク/権限/アプリ起動設計も必要。
- Cross-Region BackupはRTOをゼロにするものではない。復元時間が必要。
- バックアップとレプリケーションは違う。低RPO/低RTOならGlobal Database、Global Tables、CRRなども検討。
- バックアップを同じアカウントだけに置くと、誤削除・侵害時のリスクが残る。

---

## Related

- [Disaster Recovery Pattern](../../patterns/disaster-recovery.md)
- [Amazon EFS](efs.md)
- [Amazon FSx](fsx.md)
- [AWS Storage Gateway](storage-gateway.md)

## Official Docs

- https://docs.aws.amazon.com/aws-backup/latest/devguide/whatisbackup.html
- https://docs.aws.amazon.com/aws-backup/latest/devguide/cross-region-backup.html
- https://docs.aws.amazon.com/aws-backup/latest/devguide/create-cross-account-backup.html
