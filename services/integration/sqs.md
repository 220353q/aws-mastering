# Amazon SQS — SAP-C02重点ノート

## Overview
Amazon Simple Queue Service (SQS) は、分散システムの送信側と受信側を疎結合にするマネージドキュー。SAP-C02では、**Standard vs FIFO、visibility timeout、DLQ、Lambda連携、SNS fanout、バッファリング、非同期化** が頻出。

---

## SQSの本質

```text
Producer
  → SQS Queue
  → Consumer
```

SQSは、コンシューマが落ちてもメッセージを保持し、後から処理できる。ピーク吸収、非同期化、バックプレッシャー対策に使う。

---

## Standard Queue vs FIFO Queue

| 項目 | Standard | FIFO |
|---|---|---|
| 順序 | ベストエフォート | Message Group単位で順序保証 |
| 重複 | 少なくとも1回配信 | 重複排除機能あり |
| スループット | 高い | Standardより制約あり |
| 用途 | 一般的な非同期処理、ログ、ジョブ | 注文、決済、在庫など順序が重要な処理 |

**試験の判断**: 「順序が重要」「重複を避けたい」「同じ顧客/注文単位で順序処理」が出たらFIFOを検討。

---

## Visibility Timeout

コンシューマがメッセージを受け取ると、そのメッセージは一定時間ほかのコンシューマから見えなくなる。この時間がvisibility timeout。

```text
ReceiveMessage
  → message invisible
  → 処理成功なら DeleteMessage
  → 削除しないままtimeout超過
  → message visible again
```

処理時間より短いvisibility timeoutにすると、同じメッセージが再配信されやすくなる。

---

## Dead-Letter Queue (DLQ)

処理に何度も失敗したメッセージを隔離するキュー。

```text
Source Queue
  → maxReceiveCount超過
  → Dead-Letter Queue
```

### 使う理由

- 毒メッセージで処理が詰まるのを防ぐ
- 失敗メッセージを後から調査する
- アラームで障害検知する

---

## Lambda + SQS

SQSはLambdaのイベントソースとして使える。

```text
Producer → SQS → Lambda poller → Lambda function
```

### 設計ポイント

- Lambdaの同時実行数で処理速度を調整
- Batch sizeとpartial batch responseを検討
- visibility timeoutはLambda timeoutより長くする
- DLQまたはon-failure設計を入れる

---

## SNS + SQS Fanout

```text
SNS Topic
  ├─ SQS Queue A → Consumer A
  ├─ SQS Queue B → Consumer B
  └─ SQS Queue C → Consumer C
```

同じイベントを複数システムに配信し、それぞれが独立に処理したい場合に有効。SNSだけだと受け取り側が落ちたときの保持が弱いが、SQSを挟むと耐障害性が上がる。

---

## SQSを選ぶ要件

| 要件 | 判断 |
|---|---|
| 処理を非同期化したい | SQS |
| ピークを吸収したい | SQS |
| コンシューマ障害時もメッセージを保持したい | SQS |
| 順序と重複排除が必要 | FIFO Queue |
| イベント内容で複雑にルーティングしたい | EventBridge |
| 複数購読者へ即時通知 | SNS |
| 状態を持つワークフロー | Step Functions |

---

## よくある誤答パターン

| 誤答 | なぜ危険か |
|---|---|
| SQSでリアルタイムPush通知を作る | Push型通知はSNS/WebSocket等を検討 |
| Standard Queueで厳密順序を保証する | FIFOが必要 |
| visibility timeoutを短く設定する | 重複処理が増える |
| DLQを設定しない | 毒メッセージで再試行ループになる |
| SQSだけで複雑な分岐ワークフローを作る | Step Functions/EventBridgeと組み合わせる |

---

## SAP-C02 Focus

SQSは、**非同期化・バッファリング・疎結合・障害分離** のために選ぶ。試験では、Standard/FIFO、visibility timeout、DLQ、SNS fanout、Lambda連携の組み合わせで出る。

## Official Docs
- Visibility timeout: https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-visibility-timeout.html
- Dead-letter queues: https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-dead-letter-queues.html
- FIFO queues: https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-fifo-queues.html
