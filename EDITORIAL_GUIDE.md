# 編集規約 — AWS教材を人間が読める文章にする

この規約は、情報を減らすためではない。

読者が次の順序で理解できるよう、情報を**配置し直す**ためのものである。

```text
結論
  → 全体像
  → 仕組み
  → 判断
  → 障害
  → 詳細
  → 確認
```

---

# 1. 一ページ一問

ページタイトルは、読者が知りたい問いへ近づける。

悪い例：

- Amazon SQS
- Networking
- Database

サービス辞書では許容できるが、読み物では目的が分からない。

良い例：

- なぜQueueを挟むと障害が連鎖しにくくなるのか
- VPC接続とPrivate service公開は何が違うのか
- Multi-AZとRead Replicaをどう使い分けるか

一ページで複数の問いへ答える場合は、最初に目次と主題を示す。

---

# 2. 30秒・5分・20分の三層

## 30秒

ページ冒頭に次を置く。

- 一言の定義
- 解決する問題
- 最重要な比較対象

## 5分

- 最初の一枚
- 処理の流れ
- 選ぶ条件
- 選ばない条件

## 20分以上

- Failure behavior
- Operations
- Cost driver
- Limits
- Migration
- SAP誤答

読者が途中で止めても、中心概念を持ち帰れるようにする。

---

# 3. 良い説明の文型

AWSサービスを主語にして、抽象的な効果だけを書かない。

悪い例：

> SQSは疎結合を実現する。

良い例：

> Producerは処理要求をMessageとしてSQSへ保存する。Consumerは自分の処理速度でMessageを取得する。Consumerが一時停止してもProducerはConsumerへ直接接続しないため、処理要求をQueueに残せる。ただし即時完了ではなく、重複処理、保持期間、DLQ後の対応を設計する必要がある。

良い説明には次がある。

```text
主体
入力
処理
出力 / 状態変化
目的
条件
代償
```

---

# 4. 抽象語を単独で使わない

次の言葉は、具体化しないと意味が広すぎる。

- scalable
- resilient
- highly available
- secure
- managed
- serverless
- loosely coupled
- cost-effective
- performant
- private

## 具体化の例

### scalable

何が増えるのか。

- Request rate
- Data volume
- Connection
- Consumer数
- Throughput

### resilient

何の障害へ耐えるのか。

- Process
- Instance
- AZ
- Region
- Dependency
- Human error

### managed

何をAWSが管理し、何が利用者責任として残るのか。

---

# 5. 名詞より動詞を優先する

箱の名前を増やすより、何が起きるかを書く。

悪い文章：

> Route 53、CloudFront、ALB、ECS、Auroraを使用する。

良い文章：

> Route 53が名前に対する接続先を回答する。利用者のHTTPS requestはCloudFrontへ到達し、Cache missしたDynamic requestだけがALBへ転送される。ALBはPath ruleでECS Taskへ振り分け、TaskがAuroraへTransactionを書き込む。

動詞の候補：

- resolve
- accept
- route
- forward
- invoke
- publish
- enqueue
- poll
- persist
- replicate
- cache
- authorize
- assume
- detect
- fail over
- restore

---

# 6. 正常系の直後に障害系を書く

正常時だけ説明すると、設計判断へ使えない。

各ページで最低限、次を確認する。

```text
相手が停止したらどうなるか。
遅くなったらどこへ溜まるか。
同じ処理が二回来たらどうなるか。
一部だけ成功したらどう戻すか。
状態が壊れたら過去へ戻れるか。
```

障害系は最後の付録ではなく、仕組みの理解に含める。

---

# 7. Source of truthを書く

Dataが登場するページでは、次を区別する。

- Source of truth
- Replica
- Cache
- Backup
- Temporary state
- Audit record

例：

```text
Orderの正本: Aurora
Session: ElastiCache
Pending work: SQS
Read cache: CloudFront
Historical recovery: AWS Backup
Audit: CloudTrail / Application audit log
```

「データがある」ではなく、どの意味のDataかを書く。

---

# 8. 比較表は同じ軸で書く

悪い比較表：

| A | B |
|---|---|
| 高速 | マネージド |
| TCP | 便利 |

比較軸が揃っていない。

良い比較軸：

- 解決する問題
- Input / Output
- Routing / Processing model
- State
- Ordering
- Failure behavior
- Operations
- Cost driver
- 選ぶ条件
- 選ばない条件

比較表の後に、一文のDecision ruleを書く。

> Workを保持してConsumerの速度差を吸収するならSQS。Event内容で複数Targetへ振り分けるならEventBridge。手順と分岐を管理するならStep Functions。

---

# 9. 図を使う条件

次に該当する場合、図を使う。

- 3主体以上
- 時間順序
- 分岐
- 境界
- 正本と複製
- Control / Data plane
- Failover / Cutover

図を使わない方がよい場合：

- 一語の定義
- 二項比較の結論
- 単純な注意事項
- 料金項目の短い列挙

## 図の上限

一枚の図は原則15箱以内にする。

超える場合：

- Overview図
- Request flow図
- Security図
- Failure図

へ分割する。

---

# 10. 図の前後に文章を置く

図だけを置かない。

## 図の前

> この図は、User requestがDynamic applicationとStatic assetへ分かれる経路を示す。

## 図の後

> Static objectの正本はS3であり、CloudFrontはCacheである。Order dataの正本はAuroraである。CloudFrontはBusiness logicを実行しない。

図が何を示し、何を示していないかを書く。

---

# 11. 詳細を最初に置かない

最初から次を列挙しない。

- 全API
- 全Quota
- 全料金
- 全Integration
- 全Option

中心概念を説明した後、詳細を次へ分ける。

```text
本文: 理解と判断
比較表: 選択
サービス辞書: 詳細
公式Document: 最新値とAPI
演習: 再現
```

---

# 12. SAP向け記述

各テーマに次を追加する。

## 問題文のSignal

例：

- `minimum operational overhead`
- `fixed IP`
- `multiple subscribers`
- `near-zero downtime`

## 典型誤答

例：

- Read ReplicaをBackupとして選ぶ
- SCPで権限を付与する
- PrivateLinkでVPC全体を接続する

## 失格理由

> この選択肢はData recovery要件にRead Replicaを使っているが、Replicaは現在状態を追従し、論理削除も複製され得るため、Point-in-time recoveryを満たさない。

単に「不正解」と書かない。

---

# 13. ページテンプレート

```markdown
# 読者の問い

## 30秒要約
一言の定義、解決する問題、比較対象。

## 最初の一枚
図が必要な場合だけ置く。

## 1. 何が起きるか
主体、入力、処理、出力。

## 2. 何が嬉しいか
目的と効果。

## 3. いつ選ぶか
要件と制約。

## 4. いつ選ばないか
限界と代替。

## 5. 障害時
停止、遅延、重複、部分失敗。

## 6. State
正本、Replica、Cache、Backup。

## 7. Operations / Cost
残る責任とCost driver。

## 8. SAPでの読み方
Signal、誤答、失格理由。

## 確認問題
説明を再現させる3〜8問。

## 次に読む
比較表、サービス詳細、演習。
```

すべての項目が不要なページでは省略してよい。ただし順番はなるべく保つ。

---

# 14. レビュー基準

## Understandable

- 冒頭30秒で中心概念が分かるか
- 主語が省略されていないか
- 矢印の意味が分かるか
- Source of truthが分かるか

## Decision-ready

- 選ぶ条件があるか
- 選ばない条件があるか
- 比較対象があるか
- Trade-offがあるか

## Failure-aware

- 障害単位があるか
- 検知、Retry、Failover、Restoreを混同していないか
- Partial failureを考えたか

## Human-scale

- 一段落が長すぎないか
- 見出しだけで流れを追えるか
- 一枚の図へ詰め込みすぎていないか
- 同じ説明を複数ページへ複製していないか

## Trustworthy

- 現在値やQuotaを固定記述するときSourceがあるか
- 推測を事実のように書いていないか
- サービスの目的を過度に一般化していないか

---

# 15. 既存ページを直す順番

1. ページの問いを一文にする
2. 30秒要約を作る
3. 重複した背景説明を削る
4. FlowとStateを確認する
5. 必要なら図を一枚追加する
6. 選ぶ / 選ばないを追加する
7. 障害時を追加する
8. 詳細をサービス辞書へ逃がす
9. 確認問題を追加する
10. 次に読むページをリンクする

文章を全面的に書き直す前に、情報の順序を直す。

---

# 16. 完成の判定

良いページは、読者が次を説明できる。

```text
これは何か。
何を解決するか。
誰が何を動かすか。
状態はどこにあるか。
何と比較するか。
どの条件で選ぶか。
どこが壊れるか。
代償は何か。
```

覚えたサービス名の数ではなく、再現できる因果関係で評価する。
