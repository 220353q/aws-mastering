# Amazon EventBridge — SAP-C02重点ノート

## Overview
Amazon EventBridge は、イベント駆動アーキテクチャの中核サービス。イベントバスに届いたイベントをルールでフィルタし、Lambda、Step Functions、SQS、SNS、Event Bus、API Destinationsなどにルーティングする。

SAP-C02では、**疎結合、SaaS連携、クロスアカウントイベント、Archive/Replay、スケジュール実行、Step Functionsとの使い分け** が問われる。

---

## 基本構造

```text
Event Source
  → Event Bus
  → Rule / Event Pattern
  → Target(s)
```

EventBridgeはイベントを「保存して順番に処理するキュー」ではなく、**イベントルーター** として捉える。

---

## 主要コンポーネント

| 要素 | 役割 |
|---|---|
| Event bus | イベントを受け取り、ルールに基づいてターゲットへ配送 |
| Rule | イベントパターンまたはスケジュールでターゲットを起動 |
| Event pattern | イベント内容に基づくフィルタ |
| Target | Lambda / Step Functions / SQS / SNS / Event bus 等 |
| Archive | イベントを保存 |
| Replay | 保存したイベントを再送 |
| API Destinations | 外部HTTP APIへイベント送信 |

---

## EventBridgeを選ぶ要件

| 要件 | 判断 |
|---|---|
| サービス間を疎結合にしたい | EventBridge |
| イベント内容でルーティングしたい | EventBridge |
| SaaSや複数AWSサービスのイベントを統合したい | EventBridge |
| 失敗イベントを後で再実行したい | Archive / Replay |
| 1対多通知で単純なPub/Sub | SNSも候補 |
| 処理をバッファリングしたい | SQS |
| 複数ステップの状態管理 | Step Functions |

---

## EventBridge vs SNS vs SQS vs Step Functions

| サービス | 本質 | 選ぶ場面 |
|---|---|---|
| EventBridge | イベントルーティング | イベント内容で多様なターゲットへ振り分け |
| SNS | Pub/Sub通知 | 同じメッセージを複数購読者へ即時通知 |
| SQS | キュー/バッファ | コンシューマが自分のペースで処理 |
| Step Functions | ワークフロー状態管理 | 順序、分岐、リトライ、補償処理 |

---

## クロスアカウントイベント

中央イベントバスに複数アカウントからイベントを集約できる。

```text
App Account A → Central Event Bus
App Account B → Central Event Bus
Security Account → GuardDuty / Security Hub events
```

Organizationsやセキュリティ集約、マルチアカウント運用と相性がよい。

---

## Archive / Replay

EventBridgeはイベントをアーカイブし、後からリプレイできる。障害発生時の再処理、テスト、バグ修正後の再投入で有効。

```text
Event Bus
  → Archive
  → Replay
  → Rules / Targets
```

注意: Replayは「元のイベントバスに対して」再送する設計として理解する。SQSのような永続キューとは違う。

---

## Schedulerとの関係

定期実行はEventBridge Ruleのスケジュール、またはEventBridge Schedulerで実現できる。SAP-C02では「cron的にLambda/Step Functionsを起動したい」要件で出る。

---

## よくある誤答パターン

| 誤答 | なぜ危険か |
|---|---|
| EventBridgeをキューとして使う | バッファ/順序/バックプレッシャーはSQSを検討 |
| 複数ステップ処理をEventBridgeだけで作る | 状態管理はStep Functionsが適切 |
| 単純な大量ファンアウトに常にEventBridge | SNSの方が単純な場合がある |
| Event patternを広く書く | 不要イベントが流れ、コスト/障害範囲が増える |
| Archive/ReplayをDLQの代替と考える | 目的が異なる。ターゲット失敗処理はDLQ/Retryも検討 |

---

## SAP-C02 Focus

EventBridgeは、**イベント駆動・疎結合・マルチアカウント・SaaS連携** のキーワードで選ぶ。SQS/SNS/Step Functionsとの違いを明確にし、イベントルーティングなのか、キューなのか、Pub/Subなのか、ワークフローなのかで判断する。

## Official Docs
- Event buses: https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-event-bus.html
- Targets: https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-targets.html
- Archive and replay: https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-archive.html
