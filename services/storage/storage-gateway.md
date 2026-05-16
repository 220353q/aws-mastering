# AWS Storage Gateway

## Positioning

AWS Storage Gateway は、オンプレミス環境からAWSストレージを使うための **ハイブリッドストレージ統合サービス**。SAP-C02では「移行」だけでなく、「既存アプリを大きく変えずにS3/FSx/バックアップ/アーカイブへ接続する」文脈で出る。

---

## Gateway Types

| 種類 | 提供するインターフェース | バックエンド | 主な用途 |
|---|---|---|---|
| Amazon S3 File Gateway | NFS / SMB | S3 | オンプレファイル共有をS3へ保存 |
| Amazon FSx File Gateway | SMB | FSx for Windows File Server | オンプレからFSx Windows共有へ低レイテンシアクセス |
| Volume Gateway | iSCSI block storage | EBS Snapshot / S3 | ブロックストレージ、スナップショット、DR |
| Tape Gateway | 仮想テープライブラリ | S3 / Glacier系 | 既存バックアップソフトを維持したテープ置換 |

---

## Decision Flow

```text
オンプレアプリがNFS/SMBファイル共有を期待している？
  → S3 File Gateway / FSx File Gateway

オンプレアプリがiSCSIブロックボリュームを期待している？
  → Volume Gateway

既存バックアップソフトがテープ装置前提？
  → Tape Gateway

一回限りの高速ファイル移行？
  → DataSync

PB級でネットワーク転送が厳しい？
  → Snow Family
```

---

## S3 File Gateway

オンプレミスのNFS/SMBクライアントにファイル共有として見せ、実体をS3オブジェクトとして保存する。

### Use when

- 既存アプリを大きく変えずにS3へ保存したい
- オンプレにローカルキャッシュを持たせたい
- S3のライフサイクル、耐久性、分析基盤と連携したい

### Trap

S3 File Gatewayは「S3をファイル共有のように使わせる入口」。完全なPOSIXファイルシステム互換を求めるならEFS/FSxの方が自然な場合がある。

---

## Volume Gateway

オンプレからiSCSIブロックストレージとして使い、AWSへスナップショットを保存する。

### Use when

- ブロックストレージをオンプレアプリに見せたい
- AWS側にスナップショットを保存してDRを構成したい
- 既存アプリを改修せずバックアップ/復旧を強化したい

---

## Tape Gateway

既存バックアップソフトから仮想テープライブラリとして見せ、AWSにテープアーカイブする。

### Use when

- テープバックアップ運用を維持したい
- 物理テープ装置を廃止したい
- 長期保管・監査要件がある

---

## Storage Gateway vs DataSync vs Snow Family

| 要件 | 選ぶサービス |
|---|---|
| 継続的にオンプレからAWSストレージを使う | Storage Gateway |
| ファイル/オブジェクトを高速に移行・同期 | DataSync |
| ネットワークで移せない大容量データ | Snow Family |
| 既存SFTP/FTP/AS2クライアントを維持 | Transfer Family |

---

## SAP-C02 Focus

Storage Gatewayは、問題文に次の語があると候補になる。

- on-premises applications must continue using NFS/SMB/iSCSI/tape
- minimal application changes
- local cache
- hybrid storage
- backup to AWS while keeping existing backup software

---

## Exam Traps

- DataSyncは移行/同期に強い。Storage Gatewayは継続利用のハイブリッド入口。
- Tape Gatewayはテープ装置置換。単なるS3アーカイブではない。
- S3 File GatewayはファイルをS3オブジェクトとして保存する。S3を直接マウントする一般用途とは違う。
- Storage Gateway単体でハイブリッドネットワーク帯域問題を解決するわけではない。Direct Connect/VPNと合わせて考える。

---

## Related

- [AWS DataSync](../migration/datasync.md)
- [AWS Snow Family](../migration/snow-family.md)
- [AWS Backup](backup.md)
- [Storage Options Comparison](../../comparisons/storage-options.md)

## Official Docs

- https://docs.aws.amazon.com/storagegateway/
