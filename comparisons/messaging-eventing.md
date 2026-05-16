# EventBridge vs SNS vs SQS vs Step Functions vs Amazon MQ

SAP-C02では、Application Integration系サービスは「どれが有名か」ではなく、**通信の形**で選ぶ。

---

## 一言でいうと

| サービス | 一言 | 主な用途 |
|---|---|---|
| EventBridge | イベントルーター | イベント内容でルーティング、SaaS/クロスアカウント連携 |
| SNS | Pub/Sub通知 | 1イベントを複数購読者へ即時通知 |
| SQS | キュー/バッファ | 非同期処理、ピーク吸収、障害分離 |
| Step Functions | 状態管理ワークフロー | 順序、分岐、リトライ、補償処理 |
| Amazon MQ | 既存MQ互換 | ActiveMQ/RabbitMQ移行、プロトコル互換 |

---

## 判断フロー

```text
既存アプリがJMS/AMQP/MQTT/STOMP/OpenWire等に依存？
  └─ Yes → Amazon MQ
  └─ No
     複数ステップの状態管理、分岐、補償処理が必要？
       └─ Yes → Step Functions
       └─ No
          受信側が自分のペースで処理する必要がある？
            └─ Yes → SQS
            └─ No
               同じメッセージを複数購読者へ配りたい？
                 └─ Yes → SNS
                 └─ No
                    イベント内容で柔軟にルーティング/SaaS連携？
                      └─ Yes → EventBridge
```

---

## よくある組み合わせ

### 1. SNS + SQS Fanout

```text
SNS Topic
  ├─ SQS Queue A → Consumer A
  ├─ SQS Queue B → Consumer B
  └─ SQS Queue C → Consumer C
```

各処理が独立して失敗・再試行できる。ECサイトの注文処理、在庫、請求、通知の分離で頻出。

---

### 2. EventBridge + Step Functions

```text
Business Event
  → EventBridge Rule
  → Step Functions
  → Lambda / ECS / DynamoDB / SNS
```

イベント発生をトリガーに、複数ステップの処理を開始する。イベントルーティングと状態管理を分離できる。

---

### 3. EventBridge + SQS

```text
EventBridge
  → Rule by event pattern
  → SQS Queue
  → Worker
```

イベント内容で振り分けつつ、ワーカー処理はキューでバッファリングする。

---

### 4. Amazon MQからモダナイズ

```text
Legacy App
  → Amazon MQ
  → 新旧アプリを段階移行
```

既存アプリのプロトコル互換が最優先なら、いきなりSQS/SNS/EventBridgeへ置き換えない。

---

## 試験でのキーワード対応

| キーワード | 第一候補 |
|---|---|
| 疎結合、イベントバス、SaaS連携 | EventBridge |
| fanout、複数購読者、通知 | SNS |
| キュー、バッファ、DLQ、ワーカー | SQS |
| 順序、分岐、補償、Saga、長時間処理 | Step Functions |
| 既存MQ、ActiveMQ、RabbitMQ、プロトコル互換 | Amazon MQ |
| 順序保証、重複排除 | SQS FIFO / SNS FIFO |
| 再処理、イベント再投入 | EventBridge Archive/Replay またはSQS DLQ再処理 |

---

## 誤答パターン

| 誤答 | 正しい考え方 |
|---|---|
| EventBridgeを長期バッファとして使う | バッファはSQS |
| SQSで複数購読者fanoutを直接作る | SNS + 複数SQS |
| SNSだけで障害時の確実な保持を期待する | SQSを購読者にする |
| Step Functionsなしで複雑なSagaを組む | 状態管理はStep Functions |
| 既存MQ互換要件を無視してSQSへ移行 | Amazon MQを検討 |
| Standard SQSで厳密順序を期待する | FIFO Queueを使う |

---

## SAP-C02 Focus

Application Integrationは、**同期/非同期、push/pull、1対1/1対多、状態あり/なし、互換性要件** を見て選ぶ。問題文の「処理が詰まる」「受信側が落ちる」「複数システムに通知」「順序が必要」「既存MQを移行」がサービス選定の鍵になる。
