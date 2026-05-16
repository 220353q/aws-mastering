# Amazon DataZone

## 何をするサービスか

Amazon DataZone は、組織内外のデータをカタログ化し、発見・共有・ガバナンスするためのデータ管理サービス。SAP-C02では頻出度は高くないが、**エンタープライズデータガバナンス、データプロダクト、権限申請/承認**の文脈で重要。

## 中核概念

| 概念 | 意味 |
|---|---|
| Domain | DataZoneの管理境界 |
| Project | 利用者/チームがデータを公開・発見・利用する単位 |
| Data asset | カタログ化されたデータ資産 |
| Data product | 業務目的に沿ってまとめられたデータ資産パッケージ |
| Subscription | データ利用申請と承認の流れ |
| Environment | データを利用するためのAWS環境 |

## Lake Formation / Glueとの関係

DataZoneは、データ利用者がデータを発見し、申請し、承認されたデータを使えるようにする上位のガバナンスレイヤー。Glue Data CatalogやLake Formationと連携して、実際のメタデータや権限制御と接続する。

```text
Data Producers
   ↓ publish
Amazon DataZone Catalog
   ↓ discover / subscribe
Data Consumers
   ↓ governed access
Glue / Lake Formation / Athena / Redshift
```

## よくある誤答

- **DataZoneをETLとして選ぶ**: ETLはGlue。
- **DataZoneをBIとして選ぶ**: BIはQuickSight。
- **DataZoneをDWHとして選ぶ**: DWHはRedshift。
- **Lake Formationと完全同義だと思う**: Lake Formationはデータレイク権限制御、DataZoneは発見・共有・ガバナンスの上位体験。

## SAP-C02 Focus

- 複数部門がデータを探し、申請し、承認された範囲で使うならDataZone。
- 技術的な行/列レベルアクセス制御はLake Formation。
- メタデータカタログはGlue Data Catalog。
