# AWS AppSync

AWS AppSync は、GraphQL APIをマネージドに提供し、DynamoDB、Lambda、OpenSearch、HTTP APIなどのデータソースへ接続できるサービス。

## 一言で

GraphQL、リアルタイム更新、モバイル/Web向け柔軟APIが必要ならAppSync。

## API Gatewayとの違い

| 要件 | 選ぶ |
|---|---|
| REST/HTTP/WebSocket APIを公開 | API Gateway |
| GraphQLで複数データソースを統合 | AppSync |
| リアルタイム購読/オフライン同期寄り | AppSync |
| 単純なLambda REST API | API Gateway |

## 試験で選ぶ条件

- クライアントが必要なフィールドだけ取得したい
- 複数バックエンドをGraphQLで統合したい
- サブスクリプションによるリアルタイム更新が必要
- Cognito/IAM/OIDC/API keyで認証したい

## High-Risk Exam Traps

- GraphQL要件がなければAPI Gatewayの方が自然なことが多い。
- AppSyncはデータベースではない。DynamoDBやLambdaなどのデータソースと組み合わせる。
- Pub/Sub全般を置き換えるものではない。疎結合イベントはEventBridge/SNS/SQSも読む。

## Related

- [API Gateway](apigateway.md)
- [DynamoDB](../database/dynamodb.md)
- [Cognito](../security/cognito.md)
