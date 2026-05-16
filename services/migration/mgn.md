# AWS Application Migration Service (MGN)

## Overview
AWS Application Migration Service（MGN）は、オンプレミス、他クラウド、物理サーバー、仮想マシン上のアプリケーションを AWS へ **リホスト（lift-and-shift）** するためのサービス。ソースサーバーにエージェントを導入し、ブロックレベルで継続レプリケーションを行い、テスト起動とカットオーバーを管理する。

---

## 何を解決するサービスか

- サーバーを大きく作り替えずに EC2 へ移行したい。
- カットオーバー時の停止時間を短くしたい。
- 大量サーバーを移行ウェーブ単位で管理したい。
- 移行前にテストインスタンスを起動して検証したい。

---

## 基本アーキテクチャ

```
オンプレ / 他クラウド
  Source Server + MGN Agent
        │ 継続レプリケーション
        ▼
AWS Staging Area Subnet
  Replication Server + EBS Staging Volumes
        │ テスト / カットオーバー時に変換
        ▼
Target Subnet
  EC2 Launch Template → Test Instance / Cutover Instance
```

---

## MGN の判断軸

| 観点 | 判断 |
|---|---|
| 移行タイプ | Rehost / Lift-and-shift |
| 対象 | サーバー、VM、物理マシン、他クラウド上のインスタンス |
| データ同期 | 継続レプリケーション |
| 停止時間 | カットオーバー時のみ短時間 |
| 移行後 | EC2 として稼働。必要に応じて後でモダナイズ |

---

## DMS / DataSync との違い

| サービス | 移行対象 | 使う場面 |
|---|---|---|
| **MGN** | サーバー全体 | OS/アプリごとEC2へ移したい |
| **DMS** | DBデータ | データベースをRDS/Aurora等へ移したい |
| **DataSync** | ファイル/オブジェクト | NFS/SMB/S3/EFS/FSx間でデータ転送したい |

---

## SAP-C02 頻出シナリオ

| キーワード | 正解になりやすい選択 |
|---|---|
| `lift and shift`, `minimal changes` | MGN |
| `continuous block-level replication` | MGN |
| `test launch before cutover` | MGN |
| `migrate hundreds of servers in waves` | MGN + Migration Hub |
| `modernize after migration` | Rehost first, then refactor |

---

## 誤答パターン

- **DMSを選ぶ**: DBだけでなくOS/アプリごと移したいならDMSではない。
- **DataSyncを選ぶ**: ファイル転送には有効だが、起動可能なEC2環境は作らない。
- **Snow Familyを選ぶ**: ネットワークが使える継続レプリケーションならMGNが自然。Snowは大容量オフライン転送向き。

---

## Connections

- **Migration Hub**: 移行進捗をアプリケーション単位で追跡。
- **Application Discovery Service**: 移行前のサーバー棚卸し・依存関係把握。
- **EC2 / EBS**: 移行後の実行基盤。
- **IAM**: レプリケーション、起動、カットオーバー操作の権限管理。
- **CloudWatch**: 移行後インスタンスの監視。

---

## Well-Architected 観点

- **Reliability**: テスト起動で復旧性・依存関係を検証する。
- **Security**: 移行用ロールは最小権限。通信経路とステージング領域を保護。
- **Cost**: 移行後は Compute Optimizer や Savings Plans で最適化。
- **Operational Excellence**: 移行ウェーブ、Runbook、Rollback手順を事前定義。
