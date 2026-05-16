# Pattern: Event-Driven Architecture

## Overview
Event-driven architecture は、アプリケーション同士を直接呼び出しで密結合させず、イベントを介して疎結合にする設計パターン。SAP-C02では、EventBridge / SNS / SQS / Step Functions の使い分けが中心になる。

---

## Basic Flow

```text
Event Source
  → EventBridge / SNS / SQS
  → Lambda / Step Functions / ECS Worker
  → Target Services
```

---

## Core Services

| サービス | 役割 |
|---|---|
| Amazon EventBridge | イベントバス、イベントパターンによるルーティング、SaaS/クロスアカウント連携 |
| Amazon SNS | Pub/Sub、fanout、通知 |
| Amazon SQS | キュー、バッファリング、DLQ、ワーカー処理 |
| AWS Step Functions | 状態管理、分岐、リトライ、補償処理 |
| AWS Lambda / ECS | イベント処理の実行基盤 |
| Amazon Kinesis | 高スループットのストリーミング処理 |

---

## Reference Architecture: Order Processing

```text
Order API
  → EventBridge: OrderCreated
      ├─ Rule: inventory events → SQS InventoryQueue → Inventory Worker
      ├─ Rule: payment events → Step Functions PaymentWorkflow
      └─ Rule: notification events → SNS Topic → Email / Mobile / SQS
```

---

## Benefits

- ProducerとConsumerを疎結合にできる
- 障害範囲を分離しやすい
- ピーク負荷を吸収しやすい
- 新しいConsumerを後から追加しやすい
- EventBridge Archive/ReplayやSQS DLQにより再処理設計を入れやすい

---

## Design Decisions

| 問い | 判断 |
|---|---|
| イベント内容で配送先を変える？ | EventBridge |
| 同じメッセージを複数購読者に配る？ | SNS |
| 受信側の処理速度に合わせたい？ | SQS |
| 失敗時に隔離・再処理したい？ | SQS DLQ / EventBridge DLQ / Archive Replay |
| 複数ステップの状態管理が必要？ | Step Functions |
| 高頻度ストリーム分析？ | Kinesis / MSK |

---

## SAP Example

注文受付後に在庫更新、決済、配送、通知を行う。各処理は独立して失敗・再試行される必要があり、将来新しい処理も追加される。最適な構成は、注文イベントをEventBridgeに発行し、処理種別ごとにSQSやStep Functions、SNSへ振り分ける構成。

## Related
- [EventBridge](../services/integration/eventbridge.md)
- [SQS](../services/integration/sqs.md)
- [SNS](../services/integration/sns.md)
- [Step Functions](../services/integration/stepfunctions.md)
- [Messaging/Eventing Comparison](../comparisons/messaging-eventing.md)
