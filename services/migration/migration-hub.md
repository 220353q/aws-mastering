# AWS Migration Hub

## Overview
AWS Migration Hub は、複数のAWS移行ツールやパートナーツールによる移行状況を、アプリケーション単位で一元的に追跡するサービス。SAP-C02では、**大量サーバーを移行ウェーブ単位で管理する**、**移行進捗を可視化する**、**Application Discovery Serviceと連携する**という文脈で登場する。

---

## 役割

- 既存サーバーを発見・棚卸しする。
- サーバーをアプリケーションとしてグルーピングする。
- MGN、DMSなどの移行進捗を集約表示する。
- 移行ウェーブの計画・追跡に使う。

---

## 基本フロー

```
1. Discover
   Application Discovery Service でサーバー構成・依存関係を収集

2. Group
   サーバーをアプリケーション単位にグルーピング

3. Migrate
   MGN / DMS / DataSync などで移行

4. Track
   Migration Hub でアプリ単位の進捗を確認
```

---

## Migration Hub と各サービス

| サービス | Migration Hubでの位置づけ |
|---|---|
| **Application Discovery Service** | 移行前の情報収集 |
| **MGN** | サーバー移行の進捗連携 |
| **DMS** | DB移行の進捗連携 |
| **Migration Hub Strategy Recommendations** | 移行/モダナイズ戦略の推奨 |
| **Migration Hub Refactor Spaces** | Strangler Fig 型の段階的リファクタ支援 |

---

## SAP-C02 頻出シナリオ

| キーワード | 正解アプローチ |
|---|---|
| `track migration status across many applications` | Migration Hub |
| `group servers into applications` | Migration Hub + Discovery |
| `migration waves` | Migration Hub |
| `discover on-premises dependencies` | Application Discovery Service |
| `choose strategy for applications` | Strategy Recommendations |

---

## 誤答パターン

- **Migration Hubで実際のデータ移行を行う**と考える: Migration Hubは主に追跡・可視化。移行実行はMGN/DMS/DataSyncなど。
- **CloudWatchだけで管理する**: 個別メトリクス監視はできるが、アプリ単位の移行進捗管理にはMigration Hubが適切。

---

## Connections

- **Application Discovery Service**: サーバー情報と依存関係を収集。
- **MGN**: サーバーリホスト。
- **DMS**: DB移行。
- **AWS Organizations / Control Tower**: 移行先アカウント管理。
- **Cost Explorer / Compute Optimizer**: 移行後最適化。
