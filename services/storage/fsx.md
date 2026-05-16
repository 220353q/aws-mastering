# Amazon FSx

## Positioning

Amazon FSx は、特定ファイルシステムをフルマネージドで提供するサービス群。SAP-C02では「共有ファイル」とだけ覚えるのではなく、**どのファイルシステム互換性が必要か** で選ぶ。

---

## FSx Family

| サービス | 互換性 / 特徴 | 主な用途 |
|---|---|---|
| FSx for Windows File Server | Windows Server / SMB / AD連携 | Windowsアプリ、共有フォルダ、ホームディレクトリ |
| FSx for Lustre | 高性能並列ファイルシステム | HPC、機械学習、メディア処理、S3連携 |
| FSx for NetApp ONTAP | NetApp ONTAP互換、NFS/SMB/iSCSI | NetApp移行、スナップショット/クローン、マルチプロトコル |
| FSx for OpenZFS | OpenZFS互換、NFS | Linux/ZFS系ワークロード、低レイテンシファイル共有 |

---

## Decision Flow

```text
Windows SMB / Active Directory 連携が必要？
  → FSx for Windows File Server

HPC / ML / 高スループット / S3連携の並列処理？
  → FSx for Lustre

既存NetApp ONTAPをAWSへ移行したい？
  → FSx for NetApp ONTAP

ZFS互換、NFS、スナップショット/クローンを活かしたい？
  → FSx for OpenZFS

Linuxの一般的な共有NFSで十分？
  → EFS
```

---

## FSx for Windows File Server

### Use when

- WindowsアプリケーションがSMBファイル共有を必要とする
- Active Directory認証・Windows ACLが必要
- 既存Windowsファイルサーバーをマネージド化したい
- Multi-AZ構成で可用性を高めたい

### Trap

EFSはNFSであり、Windows SMBの代替ではない。

---

## FSx for Lustre

### Use when

- HPC、機械学習、金融シミュレーション、映像処理など高性能I/O
- S3上のデータセットと連携して高速処理
- 一時的な高性能処理基盤が必要

### Trap

Lustreは汎用ファイル共有というより、**高性能並列ファイルシステム**。通常のWebアプリ共有には過剰なことが多い。

---

## FSx for NetApp ONTAP

### Use when

- 既存オンプレNetAppをAWSへ移行
- NFS/SMB/iSCSIのマルチプロトコル要件
- ONTAP機能、スナップショット、クローン、レプリケーションを活用

---

## FSx for OpenZFS

### Use when

- Linux/ZFSベースのワークロードをAWSへ移行
- NFS互換が必要
- スナップショット/クローンや低レイテンシが重要

---

## SAP-C02 Focus

FSxの問題では、選択肢名よりも **互換性キーワード** に注目する。

| 問題文のキーワード | 選択肢 |
|---|---|
| SMB, Windows, AD, NTFS ACL | FSx for Windows File Server |
| HPC, Lustre, high performance, S3 dataset | FSx for Lustre |
| NetApp, ONTAP, SnapMirror, NFS/SMB/iSCSI | FSx for NetApp ONTAP |
| ZFS, OpenZFS, NFS, clone/snapshot | FSx for OpenZFS |

---

## Exam Traps

- 「共有ファイル」だけでEFSを選ばない。Windows/SMBならFSx for Windows。
- HPCやMLの高速ファイル処理ではEFSよりFSx for Lustreが適切なことが多い。
- 既存NetApp移行では単なるEFS移行ではなくFSx for ONTAPが自然。
- S3はファイルシステムではない。POSIX/SMB/NFS互換要件がある場合はFSx/EFSを検討。

---

## Related

- [Amazon EFS](efs.md)
- [AWS Storage Gateway](storage-gateway.md)
- [AWS DataSync](../migration/datasync.md)
- [Storage Options Comparison](../../comparisons/storage-options.md)

## Official Docs

- https://docs.aws.amazon.com/fsx/
