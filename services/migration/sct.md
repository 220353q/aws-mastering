# AWS Schema Conversion Tool / DMS Schema Conversion

## Overview
AWS Schema Conversion Tool（SCT）および DMS Schema Conversion は、異種データベース移行でスキーマやデータベースコードを移行先に合わせて変換するための機能。SAP-C02では、**DMSはデータ移行、SCT/Schema Conversionはスキーマ変換**と切り分ける。

---

## 何を変換するか

| 対象 | 例 |
|---|---|
| スキーマ | tables, indexes, constraints |
| ビュー | views |
| コードオブジェクト | stored procedures, functions, triggers |
| データ型 | Oracle NUMBER → PostgreSQL numeric など |
| 評価レポート | 自動変換できる部分 / 手動修正が必要な部分 |

---

## DMSとの役割分担

```
異種DB移行の典型:

1. SCT / DMS Schema Conversion
   Source schema を Target schema に変換
   手動修正が必要なDBオブジェクトを評価

2. DMS
   既存データを Full Load
   変更差分を CDC で継続反映

3. Cutover
   アプリ接続先を Target DB に切替
```

---

## 使う場面

| シナリオ | 判断 |
|---|---|
| Oracle → Aurora PostgreSQL | SCT/Schema Conversion + DMS |
| SQL Server → Aurora MySQL | SCT/Schema Conversion + DMS |
| MySQL → Aurora MySQL | 互換性が高いためDMS中心。SCTの必要性は低い |
| ストアドプロシージャが多い | 変換評価レポートが重要 |

---

## SAP-C02 頻出シナリオ

| キーワード | 正解アプローチ |
|---|---|
| `heterogeneous database migration` | SCT / DMS Schema Conversion + DMS |
| `schema conversion` | SCT / DMS Schema Conversion |
| `stored procedures need conversion` | SCT評価レポートで手動修正箇所を把握 |
| `Oracle to PostgreSQL` | SCT + DMS |
| `minimal downtime with schema conversion` | 事前にSCT、移行時にDMS CDC |

---

## 誤答パターン

- **DMSだけを選ぶ**: データは移せても、異種DBのスキーマやDBコードの変換が不足する。
- **SCTだけを選ぶ**: スキーマ変換はできても、本番データの継続移行にはDMSが必要。
- **MGNを選ぶ**: DBエンジンを変える移行ではなく、サーバーごと移すならMGN。

---

## Connections

- **DMS**: データ移行とCDC。
- **RDS / Aurora**: 主要なターゲットDB。
- **Schema Conversion Assessment Report**: 移行難易度・手動修正量の判断材料。
- **Migration Hub Strategy Recommendations**: 大規模移行戦略の検討。

---

## 試験の鉄則

異種DB移行は、以下の2段構えで考える。

```
Schema / Code conversion → SCT or DMS Schema Conversion
Data movement / CDC      → DMS
```
