# AWS Transfer Family

## Overview
AWS Transfer Family は、SFTP、FTPS、FTP、AS2 などの既存ファイル転送プロトコルをマネージドに提供し、転送先/転送元として Amazon S3 や Amazon EFS を利用できるサービス。SAP-C02では、**既存の取引先・顧客のSFTPクライアントを変更せずにAWSへ移行する**文脈で出やすい。

---

## 使う場面

- 取引先が既存の SFTP / FTPS / FTP クライアントを使い続ける必要がある。
- 自前のSFTPサーバー運用をやめたい。
- ファイル受領後に S3 イベント、Lambda、Step Functions で後続処理したい。
- EFS をバックエンドにして既存ファイルシステム的な利用をしたい。

---

## 基本アーキテクチャ

```
External Partners / Internal Users
        │ SFTP / FTPS / FTP / AS2
        ▼
AWS Transfer Family Server
        │
        ├── Amazon S3
        └── Amazon EFS

認証:
  Service managed users / Directory Service / Custom IdP / IAM など
```

---

## Transfer Family vs DataSync

| 観点 | Transfer Family | DataSync |
|---|---|---|
| 主目的 | ファイル転送プロトコルの受け口 | データ移行・同期 |
| 利用者 | 取引先、顧客、既存SFTPクライアント | 管理者、移行タスク |
| プロトコル | SFTP/FTPS/FTP/AS2 | NFS/SMB/S3/EFS/FSx等 |
| 継続利用 | 向いている | 移行・定期同期向き |

---

## SAP-C02 頻出シナリオ

| キーワード | 正解アプローチ |
|---|---|
| `existing SFTP clients must not change` | Transfer Family |
| `managed SFTP endpoint backed by S3` | Transfer Family |
| `B2B file transfer`, `AS2` | Transfer Family |
| `remove self-managed FTP servers` | Transfer Family |
| `NFS file server migration to S3` | DataSync |

---

## 誤答パターン

- **DataSyncを選ぶ**: ファイルサーバー移行ならDataSyncだが、外部利用者にSFTPエンドポイントを提供し続けるならTransfer Family。
- **EC2でSFTPサーバーを構築する**: 運用負荷削減が要件ならマネージドサービスを選ぶ。
- **API Gatewayを選ぶ**: API連携ではなく標準ファイル転送プロトコルが要件ならTransfer Family。

---

## Connections

- **S3 / EFS**: バックエンドストレージ。
- **IAM**: ユーザーごとのアクセス範囲制御。
- **Directory Service / Custom IdP**: 既存認証基盤との連携。
- **Lambda / EventBridge / Step Functions**: ファイル受領後の処理自動化。
- **CloudWatch Logs**: 転送ログ・監査。
