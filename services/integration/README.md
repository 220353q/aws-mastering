# Application Integration Services

## Tier 1
- **Amazon API Gateway**: API公開、認証、スロットリング、REST/HTTP/WebSocket。
- **AWS Step Functions**: 状態管理を持つワークフロー、リトライ、分岐、Saga。
- **Amazon EventBridge**: イベントバス、イベントルーティング、SaaS/クロスアカウント連携。
- **Amazon SQS**: キュー、非同期化、バッファリング、DLQ。
- **Amazon SNS**: Pub/Sub、fanout、通知。
- **Amazon MQ**: 既存ActiveMQ/RabbitMQ移行。
- **AWS App Mesh**: サービスメッシュ、mTLS、トラフィック制御。
- **AWS AppSync**: GraphQL API、リアルタイム購読、複数データソース統合。

## Design Focus
SAP-C02では、Application Integrationは単独暗記ではなく、**疎結合設計の選択問題**として問われる。

```text
イベント内容でルーティング → EventBridge
複数購読者へ通知 → SNS
処理を保持して非同期化 → SQS
順序・分岐・補償処理 → Step Functions
既存MQ互換 → Amazon MQ
API公開 → API Gateway
GraphQL/リアルタイムAPI → AppSync
```
