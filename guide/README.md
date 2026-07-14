# 人間向けAWS設計ガイド

このガイドは、AWSサービスをカテゴリ順に並べた辞書ではない。

人がシステムを理解するときの順番に沿っている。

```text
利用者は何をするか
  → Requestはどこを通るか
  → どこで処理されるか
  → 状態はどこにあるか
  → 何が壊れるか
  → どう観測し、変更し、復旧するか
  → 既存環境をどう移すか
```

---

## このガイドで身につける四つの視点

### 1. Flow — 流れ

Request、message、event、query、replicationがどこからどこへ動くか。

### 2. State — 状態

正しいデータ、Session、Queue中の未処理要求、Configurationがどこに保持されるか。

### 3. Responsibility — 責任

AWS、利用者、アプリケーション、運用担当者の誰が何を管理するか。

### 4. Failure — 失敗

何が停止し、どこまで影響し、どの仕組みで検知・切替・復旧するか。

サービス名は、この四つを実現する道具として後から置く。

---

# 本の構成

| 章 | 読者の問い | 主に扱うもの |
|---|---|---|
| [第1章](01-request-to-data.md) | クリックした後、何が起きるのか | DNS、CDN、LB、Compute、Queue、DB、Cache |
| [第2章](02-design-decisions.md) | なぜそのサービスを選ぶのか | 制約、比較、Trade-off、非機能要件 |
| [第3章](03-failure-recovery.md) | どこが壊れ、どう戻るのか | AZ/Region障害、Retry、Failover、Backup、DR |
| [第4章](04-operation-change.md) | どう安全に運用・変更するのか | Metrics、Logs、Trace、IaC、Deployment、Rollback |
| [第5章](05-migration-modernization.md) | 既存環境をどう移すのか | 7R、Wave、Data同期、Cutover、Modernization |
| [第6章](06-sap-exam.md) | SAP長文をどう解くのか | 制約語、失格条件、複数正解、誤答理由 |
| [図解アトラス](DIAGRAM_ATLAS.md) | 構成をどう描くのか | Request、Event、IAM、DR、Migrationの標準図 |

---

# 各ページの読み方

各章は、同じ順番で読めるようにする。

## 30秒要約

最初に結論だけ読む。

## 最初の一枚

関係性や時間順序がある場合は、図で全体を見る。

## 仕組み

誰が、何を、どこへ、いつ、どう動かすかを確認する。

## 判断

どの条件で選び、どの条件なら選ばないかを見る。

## 障害時

正常時だけでなく、停止・遅延・重複・部分失敗で何が起こるかを見る。

## 確認問題

説明を再現できるか確認する。

---

# 説明の粒度

## 短すぎる説明

> EventBridgeはイベント駆動サービスである。

正しいが、何も選べない。

## 適切な説明

> EventBridgeは、Producerが発行したEventを内容に応じてRuleで振り分け、複数のTargetへ配送する。Eventを長時間のWork queueとして保持することが主目的ではないため、処理速度差の吸収やWorker保護が必要ならSQSを組み合わせる。

この説明には、次がある。

- 主体
- 入力
- 処理
- 出力
- 目的
- 比較対象
- 選ばない条件

## 長すぎる説明

全API、全Quota、全料金を最初から列挙すると、中心概念が消える。

詳細は個別サービスページへ分離する。

---

# 図の役割

図は装飾ではない。文章だけでは保持しにくい関係を外部化する。

| 図が向く | 図がなくてもよい |
|---|---|
| 通信経路 | 単語の短い定義 |
| 時間順序 | 二項比較の結論 |
| 境界 | 注意事項の列挙 |
| 分岐と例外 | 一つの料金項目 |
| 正本とReplica | 単純な暗記表 |
| 正常系と障害系 | 一文で説明できる因果 |

図には、次のどれかを必ず書く。

- `HTTPS request`
- `DNS query`
- `message`
- `event`
- `SQL read/write`
- `replication`
- `AssumeRole`
- `metrics/logs`

意味のない矢印を使わない。

---

# 三つの読み方

## 初学者

1. [START_HERE](../START_HERE.md)
2. 第1章
3. 図解アトラス
4. 分からない用語だけ[説明の説明](../EXPLANATION_OF_EXPLANATIONS.md)

## 設計者

1. 第2章
2. 第3章
3. 第4章
4. [SAP設計読本](../SAP_DESIGN_READER.md)
5. `comparisons/`と`patterns/`

## SAP受験者

1. 第6章
2. [カバレッジマトリクス](../SAP_C02_COVERAGE_MATRIX.md)
3. 弱いTaskに対応する章
4. `practice/`
5. 誤答をカバレッジ表へ戻す

---

# 読み終えたかの判定

ページを読んだ後、次を口頭またはメモで説明する。

```text
この仕組みの利用者は誰か。
入力は何か。
どこで処理されるか。
状態はどこにあるか。
どこが壊れ得るか。
どの条件で選ぶか。
何とは違うか。
代償は何か。
```

答えられない項目だけ、深掘りページへ進む。
