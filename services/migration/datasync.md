# AWS DataSync

## Overview
AWS DataSync は、オンプレミス、他クラウド、AWSストレージ間でファイルまたはオブジェクトデータをオンライン転送するサービス。NFS、SMB、HDFS、Object Storage、Amazon S3、EFS、FSx などの間で、安全かつ高速にデータを移行・同期する。

---

## 使う場面

- オンプレミスの NFS / SMB ファイルサーバーを S3 / EFS / FSx に移行したい。
- 定期的にオンプレとAWS間でデータを同期したい。
- アーカイブ目的でファイルを S3 / Glacier 系ストレージに移したい。
- Snow Familyを使うほどオフライン・大容量ではなく、ネットワーク経由で転送可能。

---

## 基本アーキテクチャ

```
オンプレミス
  NFS / SMB / Object Storage
        │
        │ DataSync Agent（必要な場合）
        ▼
AWS DataSync Task
        │
        ▼
AWS Storage
  Amazon S3 / Amazon EFS / Amazon FSx
```

---

## DataSync vs Transfer Family vs Snow Family

| サービス | 主対象 | 判断軸 |
|---|---|---|
| **DataSync** | ファイル/オブジェクトのオンライン転送 | NFS/SMB/S3/EFS/FSx、移行・同期 |
| **Transfer Family** | SFTP/FTPS/FTP/AS2 のマネージド受け口 | 既存クライアント・パートナー連携維持 |
| **Snow Family** | 大容量オフライン転送 | 帯域不足、PB級、物理搬送 |

---

## SAP-C02 頻出シナリオ

| キーワード | 正解アプローチ |
|---|---|
| `NFS file server to S3` | DataSync |
| `SMB shares to FSx for Windows` | DataSync |
| `on-premises file storage to EFS` | DataSync |
| `recurring transfer`, `scheduled sync` | DataSync Task |
| `limited bandwidth`, `petabytes`, `offline` | Snow Family を検討 |
| `SFTP clients must remain unchanged` | Transfer Family |

---

## 設計ポイント

- DataSync Agent はデータソース近くに配置する。
- Direct Connect / VPN / インターネット経由の帯域とセキュリティを考慮する。
- 転送先が S3 の場合、ストレージクラス、ライフサイクル、暗号化も設計する。
- 大量小ファイルは性能に影響しやすい。移行前にファイル数・サイズ分布を確認する。
- 定期同期では削除・上書きの扱いを明確化する。

---

## Connections

- **S3 / EFS / FSx**: 主な移行先。
- **Direct Connect / VPN**: 安定した転送経路。
- **CloudWatch**: 転送メトリクス・ログ監視。
- **IAM**: S3/EFS/FSx へのアクセス権限。
- **Snow Family**: ネットワーク転送が現実的でない場合の代替。

---

## Well-Architected 観点

- **Performance**: 帯域、ファイル数、エージェント配置を最適化。
- **Security**: 転送経路、保管時暗号化、IAM権限を管理。
- **Cost**: 転送頻度、データ量、ストレージクラスを調整。
- **Reliability**: 再試行、検証、ログ監視を行う。
