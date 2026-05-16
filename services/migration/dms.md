# AWS Database Migration Service (DMS)

## Overview
AWS Database Migration Service（DMS）は、データベース、データウェアハウス、NoSQL、その他データストア間でデータを移行・継続レプリケーションするサービス。SAP-C02 では **最小停止時間のDB移行**、**同種/異種DB移行**、**CDC（Change Data Capture）** が重要。

---

## 主要コンポーネント

| コンポーネント | 説明 |
|---|---|
| **Source Endpoint** | 移行元DBへの接続情報 |
| **Target Endpoint** | 移行先DBへの接続情報 |
| **Replication Instance / DMS Serverless** | データ移行を実行する基盤 |
| **Migration Task** | Full load / CDC / Full load + CDC の移行ジョブ |
| **Replication Subnet Group** | DMS基盤を配置するサブネット |

---

## 移行モード

| モード | 説明 | 使う場面 |
|---|---|---|
| **Full Load** | 既存データを一括ロード | 停止時間を許容できる小規模移行 |
| **CDC only** | 変更データのみ継続反映 | 既に初期ロード済みの場合 |
| **Full Load + CDC** | 初期ロード後、差分を継続反映 | 最小停止時間の本番移行 |

---

## 同種移行 vs 異種移行

| 種類 | 例 | 必要なもの |
|---|---|---|
| **同種移行** | Oracle → Amazon RDS for Oracle / MySQL → Aurora MySQL | DMS中心。スキーマ互換性が高い |
| **異種移行** | Oracle → Aurora PostgreSQL / SQL Server → Aurora MySQL | DMS + Schema Conversion が必要 |

DMS は主に「データ」を移す。異種移行でテーブル定義、ビュー、ストアドプロシージャ、関数などを変換するには、DMS Schema Conversion または AWS Schema Conversion Tool を使う。

---

## 基本アーキテクチャ

```
Source DB
  Oracle / SQL Server / MySQL / PostgreSQL / etc.
        │ Full Load + CDC
        ▼
DMS Replication Instance / DMS Serverless
        │
        ▼
Target DB
  Amazon RDS / Aurora / Redshift / S3 / DynamoDB / etc.
```

---

## SAP-C02 頻出シナリオ

| キーワード | 正解アプローチ |
|---|---|
| `minimal downtime database migration` | DMS Full Load + CDC |
| `heterogeneous database migration` | DMS + Schema Conversion |
| `Oracle to Aurora PostgreSQL` | SCT/DMS Schema Conversion + DMS |
| `ongoing replication` | DMS CDC |
| `database consolidation` | DMS + target RDS/Aurora/Redshift |

---

## 誤答パターン

- **MGNを選ぶ**: サーバー全体のリホストではなくDBデータ移行ならDMS。
- **SCTだけを選ぶ**: SCTはスキーマ変換・評価が中心。データ移行にはDMSが必要。
- **Snow Familyだけを選ぶ**: 回線制約が強い大量データでは候補だが、継続CDCによる最小停止時間移行にはDMS。

---

## Connections

- **AWS SCT / DMS Schema Conversion**: 異種DBのスキーマ変換。
- **RDS / Aurora**: 主要な移行先。
- **Redshift / S3**: 分析基盤への移行先。
- **Secrets Manager**: DB接続情報の安全管理。
- **CloudWatch Logs**: タスクログ、遅延、エラー確認。
- **Migration Hub**: 移行進捗の追跡。

---

## Well-Architected 観点

- **Reliability**: CDC遅延、リトライ、整合性検証を監視。
- **Security**: DB認証情報、ネットワーク経路、暗号化を管理。
- **Performance**: Replication Instance sizing、LOB設定、並列ロードを調整。
- **Operational Excellence**: リハーサル、カットオーバー手順、ロールバック計画を用意。
