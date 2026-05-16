# AWS Application Discovery Service

## Overview
AWS Application Discovery Service は、オンプレミス環境のサーバー構成、使用率、ネットワーク依存関係などを収集し、移行計画の材料を提供するサービス。Migration Hub と連携し、サーバーをアプリケーション単位にまとめる判断材料になる。

---

## 収集する情報

| 情報 | 用途 |
|---|---|
| サーバー仕様 | CPU、メモリ、ディスク、OSなどの棚卸し |
| 使用率 | Rightsizing、移行先インスタンス選定 |
| ネットワーク接続 | アプリ依存関係、移行ウェーブ設計 |
| 実行プロセス | アプリ構成把握、不要サーバーの特定 |
| タグ/グループ | Migration Hubでのアプリ単位管理 |

---

## Agent-based vs Agentless

| 方式 | 特徴 | 使う場面 |
|---|---|---|
| **Agent-based** | サーバーにエージェントを入れて詳細情報を収集 | 詳細な依存関係・使用率が必要 |
| **Agentless** | VMware等からエージェントレスに収集 | 導入負荷を下げたい、初期棚卸し |

---

## SAP-C02 頻出シナリオ

| キーワード | 正解アプローチ |
|---|---|
| `discover on-premises servers` | Application Discovery Service |
| `dependency mapping before migration` | Application Discovery Service + Migration Hub |
| `right-size EC2 before migration` | Discovery data + Compute Optimizer / Migration Evaluator |
| `migration wave planning` | Discovery → group applications → Migration Hub |

---

## 誤答パターン

- **MGNを選ぶ**: MGNは移行実行。移行前の棚卸し・依存関係把握ならDiscovery。
- **Configを選ぶ**: ConfigはAWSリソース設定の評価。オンプレ発見ではない。
- **CloudTrailを選ぶ**: API監査であり、サーバー依存関係の発見ではない。

---

## Connections

- **Migration Hub**: Discoveryデータを表示し、アプリ単位にグループ化。
- **MGN**: 発見したサーバーのリホスト実行。
- **DMS**: 発見したDBの移行。
- **Migration Evaluator**: ビジネスケース、コスト試算。
- **Control Tower**: 移行先アカウント基盤。
