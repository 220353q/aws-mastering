# Amazon API Gateway

## 何をするサービスか

Amazon API Gatewayは、REST/HTTP/WebSocket APIの入口をマネージドで作るサービス。単なるHTTP転送ではなく、**認証/認可の入口、スロットリング、Usage Plan、リクエスト変換、監視、段階的リリース**をAPI単位で扱える。

```text
Client
  → API Gateway
      → authorizer / IAM / API key / throttling
      → Lambda / ECS / HTTP backend / AWS service
```

## API Gatewayが解決すること

| 課題 | API Gatewayでの解決 |
|---|---|
| APIの公開入口を作りたい | REST API / HTTP API / WebSocket API |
| 認証済みユーザーだけ通したい | Cognito/JWT/Lambda/IAM authorizer |
| API利用量を制限したい | Throttling / Usage Plan / API key |
| バックエンドを隠したい | Lambda/HTTP/AWS service integration |
| リクエスト形式を変換したい | Mapping / validation |
| 段階的にリリースしたい | Stage / Canary deployment |

## REST API / HTTP API / WebSocket API

| 種類 | 一言 | 選ぶ目安 |
|---|---|---|
| REST API | 機能豊富なAPI Gateway v1 | Usage Plan、API key、細かい変換、既存REST要件 |
| HTTP API | 低コスト/低レイテンシなHTTP API | Lambda/HTTP backend、JWT authorizer、シンプルAPI |
| WebSocket API | 双方向通信 | チャット、通知、リアルタイム |

## Throttling / Token Bucket

- 一言でいうと: APIに流れ込むリクエスト量を制限する仕組み。
- 何のためにあるか: API Gateway、Lambda/ECSなどのバックエンド、アカウント全体の上限を守るため。
- 何が嬉しいか: 急なburstを少し吸収しつつ、定常的なRPSを制御できる。
- 何と混同しやすいか: 認証トークン。Token BucketのtokenはJWTではなく「リクエストを1回通す券」。
- 試験問題ではどう出るか: `throttle`, `rate`, `burst`, `429 Too Many Requests`, `protect backend from spikes`。
- 間違えやすい選択肢: 認証したいのにthrottlingを選ぶ、または流量制御にCognitoだけを選ぶ。
- 小さな構成図:

```text
Token bucket
  capacity = burst
  refill rate = steady RPS

Request:
  tokenあり → pass to backend
  tokenなし → 429 / throttle
```

- 暗記のコツ、語源、語呂: **Token Bucket = 通行券のバケツ**。Cognito tokenとは別物。

主な制御単位:

| 単位 | 意味 |
|---|---|
| Account/Region | アカウントとリージョンの全体上限 |
| Stage/Route/Method | APIの環境やメソッド単位の制限 |
| Usage Plan + API Key | クライアント/契約単位のrate/quota制御 |

## API Key

- 一言でいうと: API利用者を識別し、Usage Planに紐づける文字列。
- 何のためにあるか: クライアント別の利用量制限や利用状況把握をするため。
- 何が嬉しいか: 顧客Aは100 RPS、顧客Bは10 RPS、のように制御できる。
- 何と混同しやすいか: 強い認証情報。API keyは漏洩前提で扱うべき識別子で、ユーザー認証の主役ではない。
- 試験問題ではどう出るか: `usage plan`, `quota`, `per-customer throttling`, `API key required`。
- 間違えやすい選択肢: 機密APIの認証をAPI keyだけで済ませる。
- 小さな構成図:

```text
Client with x-api-key
  → API Gateway usage plan
  → rate/quota check
  → backend
```

- 暗記のコツ、語源、語呂: **API key = 利用者番号札**。身分証明書そのものではない。

## JWT / Authorizer

- 一言でいうと: JWT authorizerは、クライアントが提示したJWTを検証してAPIへの通行可否を決める。
- 何のためにあるか: CognitoやOIDC IdPでログインしたユーザーだけをAPIへ通すため。
- 何が嬉しいか: バックエンドが毎回署名検証やissuer/audience/scope確認を実装しなくてよい。
- 何と混同しやすいか: API key。JWTは「誰か/どのscopeか」を示す認証/認可情報、API keyはusage plan用の識別子。
- 試験問題ではどう出るか: `JWT`, `OAuth scopes`, `Cognito authorizer`, `Lambda authorizer`, `IAM auth`。
- 間違えやすい選択肢: API keyでユーザーのログイン状態やscopeを判定する。
- 小さな構成図:

```text
Client
  → Cognito User Pool / OIDC IdP
  → JWT
  → API Gateway JWT authorizer
  → Lambda / backend
```

- 暗記のコツ、語源、語呂: **JWT = 署名付き通行証**。API keyは番号札。

Authorizerの選び分け:

| 要件 | 候補 |
|---|---|
| Cognito User PoolのユーザーをAPIに通す | Cognito authorizer / JWT authorizer |
| 任意のOIDC/OAuth2 JWTを検証 | HTTP API JWT authorizer |
| 独自ロジックで許可判定 | Lambda authorizer |
| AWS IAM principalだけ通す | IAM authorization / SigV4 |

## Web/API周辺との関係

| 用語 | API Gatewayとの違い |
|---|---|
| ALB | L7ロードバランサー。API管理機能はAPI Gatewayほど強くない |
| CloudFront | CDN/edge reverse proxy。キャッシュやグローバル配信が中心 |
| Nginx reverse proxy | 自前proxy。運用責任は利用者側 |
| WAF | HTTP攻撃対策。API Gatewayの前段に付けられる |
| Cognito User Pool | ユーザー認証基盤。API GatewayがJWTを検証できる |

## Connections

- **Lambda / Step Functions / ECS / HTTP backend**: APIの実行先。
- **Cognito / IAM / Lambda authorizer**: 認証・認可。
- **WAF**: Bot、SQLi、XSS、rate-based ruleなどのL7保護。
- **CloudWatch / X-Ray**: メトリクス、ログ、トレース。
- **CloudFront**: エッジ配信やカスタムドメイン構成で組み合わせる。

## Well-Architected

| Pillar | 設計ポイント |
|---|---|
| Security | JWT/IAM/Lambda authorizer、mTLS、WAF、最小権限 |
| Reliability | Throttling、バックエンド保護、段階的リリース |
| Performance | HTTP API、キャッシュ、リージョン/エッジ設計 |
| Cost | REST APIとHTTP APIの機能差・コスト差を読む |
| Operational Excellence | Access logs、metrics、X-Ray、stage管理 |

## Official Docs

- API Gateway throttling for HTTP APIs: https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-throttling.html
- API Gateway JWT authorizers: https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-jwt-authorizer.html

## SAP-C02での読み方

1. APIを「公開入口」「認証/認可」「流量制御」「バックエンド連携」に分けて読む。
2. API keyは主にUsage Plan用。ユーザー認証はCognito/JWT/Lambda authorizer/IAMで考える。
3. Token BucketのtokenはJWTではない。rate/burstを制御する通行券。
4. WAF rate-based ruleは攻撃/HTTP保護、API Gateway throttlingはAPI利用制御として読む。
5. 既存Nginx reverse proxyやALB構成は、API Gatewayへ置き換えるべき要件があるかを見る。

## このページを読んだあとに戻るべき関連ページ

- [Web Runtime and Proxy Terms](../../glossary/web-runtime.md)
- [Pool Terms](../../glossary/pool-terms.md)
- [Cognito](../security/cognito.md)
- [Access Control and Encryption Deep Dive](../../comparisons/access-control-and-encryption.md)
- [CloudFront](../networking/cloudfront.md)
- [Elastic Load Balancing](../networking/elb.md)
- [WAF](../security/waf.md)
