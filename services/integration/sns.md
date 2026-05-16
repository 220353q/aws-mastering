# Amazon SNS — SAP-C02重点ノート

## Overview
Amazon Simple Notification Service (SNS) は、Pub/Sub型のマネージド通知サービス。SAP-C02では、**fanout、SQSとの組み合わせ、Lambda/HTTP/S/email/mobile pushへの通知、フィルタポリシー、クロスアカウント通知** が問われる。

---

## SNSの本質

```text
Publisher
  → SNS Topic
     ├─ Subscriber A
     ├─ Subscriber B
     └─ Subscriber C
```

SNSは「1つのメッセージを複数の購読者へ配信する」サービス。キューではなく、Pub/Sub通知と捉える。

---

## 主なSubscriber

| Subscriber | 用途 |
|---|---|
| SQS | 非同期処理、耐障害fanout |
| Lambda | イベント発生時に関数実行 |
| HTTP/S endpoint | 外部システム通知 |
| Email / SMS | 人間向け通知 |
| Mobile push | モバイル通知 |
| Firehose | 配信/分析基盤への転送 |

---

## SNS + SQS Fanout

最も重要なSAP-C02パターン。

```text
OrderCreated Event
  → SNS Topic
     ├─ SQS Queue: Billing
     ├─ SQS Queue: Inventory
     └─ SQS Queue: Notification
```

各システムが独立したキューを持つため、1つの消費者障害が他システムへ波及しにくい。

---

## SNS Filtering

購読側にfilter policyを設定し、必要なメッセージだけを受け取れる。

```text
SNS Topic: all events
  ├─ Subscription A: type = order
  └─ Subscription B: type = payment
```

単純な属性ベースの振り分けならSNS filteringで足りる。より複雑なイベントパターン、SaaS連携、マルチアカウントイベントルーティングならEventBridgeを検討する。

---

## SNS FIFO

SNSにもFIFO Topicがある。SQS FIFO Queueと組み合わせることで、順序性と重複排除を考慮したfanoutが可能。

```text
SNS FIFO Topic
  → SQS FIFO Queue
```

順序が重要なイベント配信ではStandard TopicではなくFIFO Topicを検討する。

---

## SNSを選ぶ要件

| 要件 | 判断 |
|---|---|
| 1イベントを複数購読者へ通知したい | SNS |
| 受信側ごとに処理を独立させたい | SNS + SQS |
| 人間や外部HTTP endpointへ通知したい | SNS |
| モバイルPush通知 | SNS |
| イベント内容で複雑ルーティング | EventBridge |
| 処理を保持して順番に消費 | SQS |

---

## EventBridgeとの使い分け

| 観点 | SNS | EventBridge |
|---|---|---|
| 得意 | Pub/Sub fanout | イベントルーティング |
| フィルタ | Message attributes中心 | JSONイベントパターン |
| SaaS連携 | 限定的 | 得意 |
| Archive/Replay | 基本機能ではない | あり |
| 単純通知 | 得意 | やや重い場合あり |

---

## よくある誤答パターン

| 誤答 | なぜ危険か |
|---|---|
| SNSだけでメッセージ保持を期待する | 受信側耐障害性にはSQSを組み合わせる |
| 複雑なイベントルーティングをSNSで無理に作る | EventBridgeが適切な場合がある |
| 順序が重要なのにStandard Topicを使う | FIFO Topic / FIFO Queueを検討 |
| 1つのSQS Queueを複数処理で共有する | 処理独立性が下がる。fanoutでは各処理にQueueを分ける |
| 人間通知とシステム連携を同じ設計で扱う | 再試行・DLQ・監査要件が異なる |

---

## SAP-C02 Focus

SNSは、**Pub/Sub、fanout、通知** のキーワードで選ぶ。信頼性のあるfanoutではSNS単体ではなく、SNS + SQSの組み合わせを第一候補にする。

## Official Docs
- SNS to SQS fanout: https://docs.aws.amazon.com/sns/latest/dg/sns-sqs-as-subscriber.html
- SNS to Lambda: https://docs.aws.amazon.com/sns/latest/dg/sns-lambda-as-subscriber.html
- Mobile push notifications: https://docs.aws.amazon.com/sns/latest/dg/mobile-push-notifications.html
