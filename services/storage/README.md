# Storage Services

## Tier 1

| サービス | 詳細 | 主な用途 |
|---|---|---|
| Amazon S3 | [s3.md](s3.md) | オブジェクトストレージ、データレイク、静的配信、CRR |
| Amazon EBS | 未作成 | EC2向けブロックストレージ、スナップショット、性能設計 |
| Amazon EFS | [efs.md](efs.md) | Linux/NFS共有ファイル、マルチAZ、コンテナ/Lambda連携 |
| Amazon FSx | [fsx.md](fsx.md) | Windows SMB、Lustre、ONTAP、OpenZFS |
| AWS Storage Gateway | [storage-gateway.md](storage-gateway.md) | ハイブリッドストレージ、File/Volume/Tape Gateway |
| AWS Backup | [backup.md](backup.md) | 一元バックアップ、クロスリージョン/クロスアカウント、監査 |

## Tier 2/3

- Amazon S3 Glacier
- AWS Snow Family: 詳細は [migration/snow-family.md](../migration/snow-family.md)
- AWS DataSync: 詳細は [migration/datasync.md](../migration/datasync.md)

## Key Decision

- **S3**: オブジェクト/API/データレイク
- **EBS**: EC2専用ブロック
- **EFS**: Linux/NFS共有
- **FSx**: Windows/HPC/NetApp/ZFS互換
- **Storage Gateway**: オンプレからAWSストレージを継続利用
- **AWS Backup**: バックアップ統制と監査

詳しくは [Storage Options Comparison](../../comparisons/storage-options.md) を参照。
