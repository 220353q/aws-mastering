# Comparison: AWS Storage Options

## Purpose

SAP-C02では、ストレージ問題は「容量」よりも **アクセスパターン、プロトコル、共有要件、可用性、移行方式** で判断する。

---

## Service Selection Table

| 要件 | 選ぶサービス | 理由 |
|---|---|---|
| オブジェクト、静的ファイル、データレイク | S3 | 高耐久・ライフサイクル・イベント連携 |
| EC2専用の低レイテンシブロック | EBS | ブロックデバイス、IOPS/Throughput制御 |
| 複数LinuxホストからNFS共有 | EFS | マネージドNFS、容量自動伸縮 |
| Windows SMB / AD連携 | FSx for Windows File Server | SMB/Windows ACL/AD互換 |
| HPC/MLの高性能並列ファイル | FSx for Lustre | 高スループット、S3連携 |
| 既存NetApp移行 | FSx for NetApp ONTAP | ONTAP互換、NFS/SMB/iSCSI |
| オンプレからNFS/SMB/iSCSI/テープ継続利用 | Storage Gateway | ハイブリッド入口、ローカルキャッシュ |
| ファイル/オブジェクト移行・同期 | DataSync | オンライン転送に最適 |
| ネットワークで移せない大容量 | Snow Family | オフライン転送 |
| 複数サービスのバックアップ統制 | AWS Backup | バックアップポリシー、クロスアカウント/リージョン |

---

## Decision Flow

```text
アプリがオブジェクトAPIでよい？
  → S3

ブロックデバイスが必要？
  → EBS

複数ホストからファイル共有？
  → Linux/NFSなら EFS
  → Windows/SMBなら FSx for Windows
  → HPCなら FSx for Lustre
  → NetApp互換なら FSx for ONTAP

オンプレアプリを変えずAWSストレージを使いたい？
  → Storage Gateway

データ移行・同期が主目的？
  → DataSync / Snow Family

保護・保持・監査が主目的？
  → AWS Backup
```

---

## Common Exam Traps

- 「共有ストレージ」だけでEFSに飛ばない。Windows/SMBならFSx。
- S3はファイルシステムではない。POSIX/SMB/NFS互換要件があるならEFS/FSx。
- Storage Gatewayは継続的なハイブリッド利用、DataSyncは移行/同期。
- Backupはバックアップ管理であり、アクティブ/アクティブDRではない。
- Glacier系はアーカイブ。即時共有ファイル用途ではない。

---

## Related Services

- [Amazon S3](../services/storage/s3.md)
- [Amazon EFS](../services/storage/efs.md)
- [Amazon FSx](../services/storage/fsx.md)
- [AWS Storage Gateway](../services/storage/storage-gateway.md)
- [AWS Backup](../services/storage/backup.md)
- [AWS DataSync](../services/migration/datasync.md)
- [AWS Snow Family](../services/migration/snow-family.md)
