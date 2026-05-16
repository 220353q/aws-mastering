# Amazon EBS

Amazon Elastic Block Store (EBS) は、EC2インスタンスにアタッチして使うブロックストレージ。SAP-C02では、S3/EFS/FSxとの違い、IOPS/throughput設計、スナップショット、Multi-AZではない点が頻出。

## 一言で

EC2に低レイテンシのブロックデバイスが必要ならEBS。複数ホスト共有ならEFS/FSx、オブジェクトならS3。

## 試験で選ぶ条件

- EC2のOSディスク、データディスクが必要
- ランダムI/O、低レイテンシ、IOPS制御が必要
- スナップショットでバックアップ/複製したい
- gp3/io2などで性能とコストを調整したい

## 重要ポイント

| 項目 | 試験での見方 |
|---|---|
| Availability Zone | EBSボリュームは基本的に単一AZ内。別AZのEC2へ直接アタッチしない |
| Snapshot | S3に保存される増分バックアップ。リージョンコピー可能 |
| gp3 | 汎用。容量とIOPS/throughputを分離して調整しやすい |
| io2 / io2 Block Express | 高IOPS、ミッションクリティカルDB向け |
| st1/sc1 | スループット重視HDD。ブートボリュームには使わない |
| Encryption | KMS連携。スナップショットや派生ボリュームにも影響 |

## EBS / EFS / S3 / FSx

| 要件 | 選ぶ |
|---|---|
| EC2専用ブロックデバイス | EBS |
| 複数LinuxホストからNFS共有 | EFS |
| Windows SMB / AD連携 | FSx for Windows |
| オブジェクト、データレイク、静的配信 | S3 |

## High-Risk Exam Traps

- EBSをMulti-AZ共有ストレージとして扱わない。
- S3をブロックデバイスとしてマウントする設計を正解にしない。
- スナップショットはバックアップであり、即時Active/Active共有ではない。
- 暗号化スナップショットの共有ではKMS key policyも確認する。

## Related

- [EC2](../compute/ec2.md)
- [S3](s3.md)
- [EFS](efs.md)
- [FSx](fsx.md)
- [Storage Options](../../comparisons/storage-options.md)
