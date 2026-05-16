# Pool Terms

`Pool`は、同じ種類のものをまとめて管理し、必要なときに取り出したり、割り当てたり、再利用したりする「貯め場」。AWS学習では、User Pool、Identity Pool、Connection Pool、Token Bucket、Thread Poolが混ざりやすい。

## Poolの共通意味

```text
Pool
  = collection managed as a group
  = 必要なときに取り出す
  = 数や割当を制御する
  = 使い終わったら戻す場合もある
```

ただし、何を貯めるかは用語ごとに全く違う。

| 用語 | Poolしているもの | 何を解決するか |
|---|---|---|
| User Pool | アプリユーザー | ログイン/MFA/JWT発行 |
| Identity Pool | IDとIAM Roleの対応 | AWS一時認証情報の発行 |
| Connection Pool | DB接続 | 接続確立コストとDB接続数 |
| Token Bucket | リクエストを通す券 | APIスロットリング |
| Thread Pool | 実行スレッド | 同時処理数の制御 |

## Cognito User Pool

- 一言でいうと: アプリユーザーの台帳。
- 何のためにあるか: ユーザー登録、ログイン、MFA、パスワード、JWT発行を管理するため。
- 何が嬉しいか: アプリで認証基盤を自作せずに済む。
- 何と混同しやすいか: Identity Pool。User PoolはJWTを出すが、AWS API用の一時クレデンシャルは出さない。
- 試験問題ではどう出るか: `customer login`, `MFA`, `social login`, `JWT`, `API Gateway authorizer`。
- 間違えやすい選択肢: User PoolだけでS3へ直接PutObjectできると思う。
- 小さな構成図:

```text
User
  → Cognito User Pool
  → JWT
  → API Gateway / ALB / AppSync
```

- 暗記のコツ、語源、語呂: **User Pool = 人の台帳**。

## Cognito Identity Pool

- 一言でいうと: 認証済みユーザーをAWS一時認証情報へ交換する場所。
- 何のためにあるか: モバイル/ブラウザアプリからS3やDynamoDBなどのAWS APIへ制限付きで直接アクセスさせるため。
- 何が嬉しいか: サーバーを経由しないアップロードなどを、IAM Roleで最小権限化できる。
- 何と混同しやすいか: User Pool。Identity Poolはログイン台帳ではなく、STS credentialsへの交換所。
- 試験問題ではどう出るか: `mobile app directly uploads to S3`, `temporary AWS credentials`, `authenticated and unauthenticated roles`。
- 間違えやすい選択肢: Identity Poolだけでパスワード/MFA/JWTを管理すると考える。
- 小さな構成図:

```text
User Pool / OIDC / SAML / Social IdP
  → Identity Pool
  → STS temporary credentials
  → S3 / DynamoDB
```

- 暗記のコツ、語源、語呂: **Identity Pool = AWS権限への交換所**。

## Connection Pool

- 一言でいうと: 作成済みDB接続の使い回し箱。
- 何のためにあるか: DB接続を毎回作るコストを避けるため。
- 何が嬉しいか: TLSハンドシェイク、認証、メモリ、CPU、最大接続数の負荷を減らせる。
- 何と混同しやすいか: データキャッシュ。Connection Poolはクエリ結果を保存しない。
- 試験問題ではどう出るか: `Lambda causes too many database connections`, `connection storm`, `RDS Proxy`。
- 間違えやすい選択肢: 頻繁に読む値を速くするためにConnection Poolを選ぶ。
- 小さな構成図:

```text
Without pool:
  Request → open DB connection → query → close

With pool:
  Request → borrow existing connection → query → return
```

- 暗記のコツ、語源、語呂: **Connection Pool = DB接続の使い回し**。

## RDS Proxy Pool

- 一言でいうと: AWSマネージドのRDS/Aurora向け接続プール。
- 何のためにあるか: Lambdaなどの急激な同時実行でDB接続が枯渇するのを防ぐため。
- 何が嬉しいか: 接続再利用、接続数制御、Secrets Manager/IAM認証、フェイルオーバー耐性。
- 何と混同しやすいか: ElastiCache。RDS Proxyは接続、ElastiCacheはデータをキャッシュする。
- 試験問題ではどう出るか: `serverless`, `unpredictable surges`, `pool and share database connections`。
- 間違えやすい選択肢: RDS Proxyでクエリ結果をキャッシュする。
- 小さな構成図:

```text
Lambda / ECS / App
  → RDS Proxy
      → smaller pool of DB connections
          → RDS / Aurora
```

- 暗記のコツ、語源、語呂: **ProxyがDBの前で接続をさばく**。

## Token Bucket

- 一言でいうと: リクエストを通すための券を貯めるバケツ。
- 何のためにあるか: APIを過負荷から守り、急なburstと定常rateを制御するため。
- 何が嬉しいか: 短時間の急増は少し許しつつ、長期的な流量を制限できる。
- 何と混同しやすいか: JWT/access token。Token Bucketのtokenは認証トークンではなく「通行券」の比喩。
- 試験問題ではどう出るか: `throttling`, `burst`, `rate`, `429 Too Many Requests`, `API Gateway`。
- 間違えやすい選択肢: API keyやJWTをToken Bucketのtokenと同一視する。
- 小さな構成図:

```text
Bucket capacity = burst
Refill rate     = steady RPS

Request:
  tokenあり → pass
  tokenなし → throttle / 429
```

- 暗記のコツ、語源、語呂: **Token Bucket = リクエスト券のバケツ**。

## Thread Pool

- 一言でいうと: 処理を実行するスレッドの作業員リスト。
- 何のためにあるか: 同時処理数を制御し、サーバーを過負荷から守るため。
- 何が嬉しいか: リクエストごとに無限にスレッドを作らず、一定数で処理を回せる。
- 何と混同しやすいか: Connection Pool。Thread Poolは実行係、Connection PoolはDB接続。
- 試験問題ではどう出るか: AWSサービス名として直接より、Tomcat/Nginx/アプリサーバーの前提知識として出る。
- 間違えやすい選択肢: DB接続数問題をThread Poolだけで解決できると考える。
- 小さな構成図:

```text
Requests
  → queue
  → fixed worker threads
  → application logic
```

- 暗記のコツ、語源、語呂: **Thread Pool = 実行係の人数管理**。

## 混同しない比較表

| 問題文 | 何を聞いているか | 選ぶ |
|---|---|---|
| customer login, MFA, JWT | 認証 | User Pool |
| browser/mobile app gets AWS credentials | AWS API認可 | Identity Pool |
| Lambda overwhelms RDS connections | 接続数 | RDS Proxy / Connection Pool |
| API requests exceed configured rate | 流量制御 | API Gateway throttling / Token Bucket |
| app server cannot handle many concurrent requests | 実行係 | Thread Pool / Auto Scaling / Queue |
| repeated reads need low latency | データキャッシュ | ElastiCache / DAX |

## SAP-C02での読み方

1. Poolという単語だけで判断しない。何をpoolしているかを見る。
2. User Poolは認証、Identity PoolはAWS credentials、Connection PoolはDB接続。
3. Token BucketのtokenはJWTではない。
4. 接続問題、認証問題、流量問題、実行並列数問題を分けて選択肢を切る。

## このページを読んだあとに戻るべき関連ページ

- [Amazon Cognito](../services/security/cognito.md)
- [Amazon API Gateway](../services/integration/apigateway.md)
- [RDS / Aurora Connection Deep Dive](../comparisons/rds-aurora-connection-deep-dive.md)
- [Web Runtime and Proxy Terms](web-runtime.md)
- [Access Control and Encryption Deep Dive](../comparisons/access-control-and-encryption.md)

