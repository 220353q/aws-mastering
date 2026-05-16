# Web Runtime and Proxy Terms

AWS問題文では、Tomcat、Nginx、Web server、Application server、Reverse proxyのような一般Web用語が説明なしに出る。ここで大事なのは製品名の暗記ではなく、**HTTPを誰が受け、アプリコードを誰が実行し、後ろへ誰が転送するか**を分けること。

## Webリクエストの基本形

```text
Browser / Mobile app
  → DNS
  → CDN / reverse proxy / load balancer
  → web server
  → application server
  → database / cache / external API
```

現実の構成では、1つのソフトウェアが複数の役割を持つことがある。NginxはWeb serverにもreverse proxyにもなる。ALBはAWSマネージドのL7 reverse proxy/load balancerとして読める。

## Web Server

- 一言でいうと: HTTPを受け、静的ファイルを返したり後ろへ転送したりする入口。
- 何のためにあるか: ブラウザからのHTTP/HTTPSリクエストを扱うため。
- 何が嬉しいか: 静的ファイル配信、TLS終端、圧縮、アクセスログ、proxyができる。
- 何と混同しやすいか: Application server。Web serverは主にHTTP入口、Application serverはアプリコード実行。
- 試験問題ではどう出るか: `static assets`, `Apache/Nginx web tier`, `TLS termination`, `reverse proxy`。
- 間違えやすい選択肢: 静的ファイル配信にEC2を残し続け、S3 + CloudFrontを検討しない。
- 小さな構成図:

```text
Client
  → Nginx / Apache HTTP Server
  → static file or proxy to app
```

- 暗記のコツ、語源、語呂: **Web server = HTTP受付**。

## Application Server

- 一言でいうと: アプリケーションコードを実行するサーバー。
- 何のためにあるか: ログイン処理、注文処理、DBアクセスなどの動的処理を動かすため。
- 何が嬉しいか: Java、Python、Ruby、Node.jsなどのアプリを実行できる。
- 何と混同しやすいか: Web server。静的ファイルを返すだけならApplication serverは不要な場合がある。
- 試験問題ではどう出るか: `business logic`, `legacy application`, `session state`, `application tier`。
- 間違えやすい選択肢: ALBやCloudFrontがアプリコードを実行すると考える。
- 小さな構成図:

```text
ALB
  → App server on EC2/ECS/EKS
      → business logic
      → RDS / DynamoDB / ElastiCache
```

- 暗記のコツ、語源、語呂: **Application server = コード実行係**。

## Tomcat

- 一言でいうと: Java Webアプリを動かすApplication server/Servlet container。
- 何のためにあるか: Java Servlet、JSP、Spring系アプリを実行するため。
- 何が嬉しいか: 既存Javaアプリを比較的そのまま動かせる。
- 何と混同しやすいか: NginxやALB。Tomcatはロードバランサーではなく、Javaアプリの実行場所。
- 試験問題ではどう出るか: `legacy Java application on Tomcat`, `rehost`, `Elastic Beanstalk`, `containerize`。
- 間違えやすい選択肢: Tomcat依存の既存アプリをいきなりLambdaへ移す。
- 小さな構成図:

```text
ALB
  → EC2 / ECS task
      → Tomcat
          → Java Web App
```

- 暗記のコツ、語源、語呂: **Tomcat = Java Webコンテナ**。Servlet/JSP/Spring系アプリの実行場所として覚える。

## Nginx

- 一言でいうと: Web server、reverse proxy、軽量ロードバランサーとして使われるソフトウェア。
- 何のためにあるか: HTTP入口をさばき、静的配信や後段アプリへの転送を行うため。
- 何が嬉しいか: 高速、設定しやすい、TLS終端やpath routingができる。
- 何と混同しやすいか: Tomcat。Nginxは入口/proxy、TomcatはJavaアプリ実行。
- 試験問題ではどう出るか: `Nginx reverse proxy`, `static content`, `proxy to backend`, `ingress`。
- 間違えやすい選択肢: NginxがあるからALB/CloudFront/API Gatewayが不要と決めつける。
- 小さな構成図:

```text
Client
  → Nginx
      ├─ /static → local files
      └─ /api    → app server
```

- 暗記のコツ、語源、語呂: **Nginx = engine-x = HTTPのエンジン役**。

## Reverse Proxy

- 一言でいうと: クライアントの代わりに後ろのサーバーへリクエストを渡す入口。
- 何のためにあるか: 後段サーバーを隠し、TLS終端、ルーティング、認証、負荷分散を入口で行うため。
- 何が嬉しいか: クライアントは入口だけ知ればよく、後段を入れ替えやすい。
- 何と混同しやすいか: Forward proxy。Forward proxyはクライアント側の代理、reverse proxyはサーバー側の代理。
- 試験問題ではどう出るか: `facade`, `strangler fig`, `path-based routing`, `central API entry point`。
- 間違えやすい選択肢: Reverse proxyをアプリ認可やDB接続プールの代替にする。
- 小さな構成図:

```text
Client
  → Reverse Proxy
      ├─ /orders → Orders service
      └─ /users  → Users service
```

- 暗記のコツ、語源、語呂: **Reverse = サーバー側にいる代理**。

## AWSサービスへの読み替え

| 一般用語 | AWSでの代表 | 読み替え |
|---|---|---|
| CDN / edge reverse proxy | CloudFront | 世界中の入口、キャッシュ、OAC、WAF連携 |
| L7 reverse proxy / LB | ALB | path/host routing、HTTP/HTTPS負荷分散 |
| API facade | API Gateway | REST/HTTP/WebSocket、認証、スロットリング |
| Web server on VM | EC2 + Nginx/Apache | 既存構成のリホスト |
| App server | EC2/ECS/EKS/Beanstalk | アプリコードの実行場所 |
| Java app server | Tomcat on EC2/ECS/Beanstalk | 既存Javaの移行先 |
| Kubernetes ingress | ALB Controller / Nginx Ingress | EKS内外の入口制御 |

## よくある混同

| 混同 | 正しい見方 |
|---|---|
| Tomcat vs Nginx | TomcatはJavaアプリ実行、NginxはHTTP入口/proxyが中心 |
| ALB vs API Gateway | ALBは汎用L7 LB、API GatewayはAPI管理/認証/スロットリングが強い |
| CloudFront vs Global Accelerator | CloudFrontはHTTPキャッシュ/CDN、Global Acceleratorは固定Anycast IPでTCP/UDPにも向く |
| Reverse proxy vs NAT | Reverse proxyはアプリ入口、NATはIP変換の外向き出口 |
| TLS終端 vs 認証 | TLSは通信暗号化、認証は相手が誰かの確認 |

## SAP-C02での読み方

| 問題文 | 読み替え | 選びやすい候補 |
|---|---|---|
| legacy Java app on Tomcat | 既存Java実行環境を保つ | EC2, Elastic Beanstalk, ECS/EKS |
| Nginx reverse proxy with path routing | L7入口をマネージド化できるか | ALB / API Gateway / CloudFront |
| static assets served by web server | 静的配信を分離 | S3 + CloudFront |
| session state on app server | ステート外出し | ElastiCache / DynamoDB |
| API rate limit / auth / usage plan | API管理 | API Gateway |
| global low-latency HTTP cache | エッジキャッシュ | CloudFront |

## このページを読んだあとに戻るべき関連ページ

- [Amazon API Gateway](../services/integration/apigateway.md)
- [Amazon CloudFront](../services/networking/cloudfront.md)
- [Elastic Load Balancing](../services/networking/elb.md)
- [Strangler Fig Pattern](../patterns/strangler-fig.md)
- [Pool Terms](pool-terms.md)
