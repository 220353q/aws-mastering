# Protocols and Load Balancer Positioning — ALB / NLB はどこに置くのか

## 何のための資料か

AWS設計では、HTTP/HTTPS、WebSocket、MQTT、gRPC、TCP、UDPなどの通信方式と、CloudFront、API Gateway、ALB、NLB、IoT Core、Transfer Familyなどの入口サービスを混同しやすい。

さらに、ALBやNLBは「サービスの前段」に置くことが多いが、何の前に置くのか、なぜ置くのか、置くことで何が良くなるのかを理解していないと、選択肢を見たときに迷う。

この資料では、プロトコルごとの入口サービスと、ALB/NLBが各サービスの前後に来るパターンを具体例で整理する。

---

## まず結論

```text
CloudFront = HTTP/HTTPSのエッジ配信・キャッシュ・保護
API Gateway = API管理の入口
ALB = HTTP/HTTPS/gRPC/WebSocketなどL7アプリ通信の前段
NLB = TCP/TLS/UDPなどL4通信の前段
IoT Core = MQTT/HTTPS/WebSocketによるIoTデバイス通信
Transfer Family = SFTP/FTPS/FTP/AS2のマネージドファイル転送入口
Global Accelerator = TCP/UDP入口のグローバル最適化
```

---

## 1. プロトコル別の入口サービス

| 通信方式 | 代表的な入口 | 向く用途 |
|---|---|---|
| HTTP / HTTPS | CloudFront, API Gateway, ALB, Lambda Function URL | Web, API, 静的/動的コンテンツ |
| REST / HTTP API | API Gateway, ALB | API公開、外部システム連携 |
| WebSocket | API Gateway WebSocket API, ALB, CloudFront中継, IoT Core | リアルタイム双方向通信 |
| MQTT / MQTT over WSS | AWS IoT Core | IoT Pub/Sub、デバイス通信 |
| gRPC | ALB | マイクロサービス間RPC、HTTP/2 |
| TCP | NLB, Global Accelerator | 非HTTPアプリ、SFTP、自前プロトコル |
| UDP | NLB, Global Accelerator | ゲーム、VoIP、低遅延通信 |
| SFTP / FTPS / FTP / AS2 | AWS Transfer Family | B2Bファイル転送、既存クライアント維持 |

---

## 2. ALBとNLBの最短比較

既存のELBノートでは、ALBはHTTP/HTTPS、パス/ホストベースルーティング、コンテナ向け、NLBはTCP/UDP/TLS、固定IP、高性能、PrivateLink公開向けとして整理している。

| 観点 | ALB | NLB |
|---|---|---|
| レイヤー | L7 | L4 |
| 主な通信 | HTTP, HTTPS, WebSocket, HTTP/2, gRPC | TCP, TLS, UDP |
| 見るもの | Host, Path, Header, Methodなど | IP, Port, Connection |
| ルーティング | Host/path/header条件 | Port/protocol中心 |
| 認証連携 | OIDC/Cognito連携が可能 | 基本はL4。アプリ側で認証 |
| WAF | 関連付け可能 | 直接は基本対象外 |
| 固定IP | 直接は苦手。GA等を検討 | 得意 |
| PrivateLink提供側 | 通常はNLB | 頻出 |
| 代表ターゲット | EC2, ECS, EKS, Lambda, IP | EC2, ECS, EKS, IP, ALBなど |

---

## 3. 「前段に置く」とは何か

ALB/NLBを前段に置くとは、クライアントから来た通信をまずLoad Balancerで受け、その後ろの複数ターゲットへ振り分けること。

```text
Client
  → Load Balancer
  → Target Group
  → Service instances
```

前段に置くことで得られるもの。

```text
単一の入口DNS名
ヘルスチェック
正常なターゲットだけに転送
複数AZへの分散
スケールアウトしたターゲットへの分散
証明書/TLS終端の集約
ルーティング条件の集約
障害時の切り離し
```

---

## 4. ALBがサービスの前段に来る代表パターン

### 4.1 ALB → EC2 Webアプリ

```text
Users
  → ALB
  → EC2 Auto Scaling Group
```

向く要件。

```text
HTTP/HTTPS Webアプリ
複数EC2へ分散したい
Auto Scalingしたい
パス/ホストベースで振り分けたい
TLS証明書をALBで管理したい
```

何が良いか。

```text
EC2を直接公開しなくてよい
正常なEC2だけへ流せる
AZ障害に強くなる
TLS終端を集約できる
Auto Scalingと相性が良い
```

---

### 4.2 ALB → ECS Service

```text
Users
  → ALB
  → ECS Service
      ├─ task A
      ├─ task B
      └─ task C
```

向く要件。

```text
コンテナWebアプリ
タスク数が増減する
blue/greenやrolling deploymentをしたい
/api と /admin などでサービスを分けたい
```

何が良いか。

```text
ECSタスクのIP/port変化を吸収できる
target groupでタスクを自動登録/解除できる
path-based routingで複数マイクロサービスに分けられる
デプロイ時にヘルスチェックで安全に切り替えられる
```

---

### 4.3 ALB → EKS / Kubernetes Ingress

```text
Users
  → ALB
  → Kubernetes Ingress
  → Pods / Services
```

向く要件。

```text
Kubernetes上のHTTP/HTTPSサービスを公開したい
Ingressでpath/host routingしたい
AWS Load Balancer Controllerを使いたい
```

何が良いか。

```text
KubernetesのIngressとAWSのALBを統合できる
HTTPレイヤーのルーティングをALBに任せられる
WAFやACMと組み合わせやすい
```

---

### 4.4 ALB → Lambda

```text
Users
  → ALB
  → Lambda target
```

向く要件。

```text
既存ALB配下に一部だけLambda処理を置きたい
HTTPイベントをLambdaへ流したい
API GatewayほどのAPI管理は不要
```

何が良いか。

```text
既存Web入口を保ったままserverless処理を混ぜられる
path routingで一部パスだけLambdaへ流せる
```

注意。

```text
API Key、Usage Plan、細かいAPI管理が必要ならAPI Gatewayを検討する。
```

---

### 4.5 CloudFront → ALB → ECS/EC2

```text
Users
  → CloudFront
  → ALB
  → ECS / EC2
```

向く要件。

```text
グローバル配信したい
WAFをedgeに置きたい
静的/動的コンテンツを高速化したい
ALBを直接公開したくない
```

何が良いか。

```text
ユーザーに近いedgeで受けられる
キャッシュできるものはオリジン負荷を下げられる
WAF/Shield/ACMと組み合わせやすい
ALBをoriginとしてアプリへ流せる
```

注意。

```text
CloudFrontはCDN/edgeの前段。
ALBはアプリケーション負荷分散。
役割が違うので、CloudFrontがALBを完全代替するわけではない。
```

---

## 5. NLBがサービスの前段に来る代表パターン

### 5.1 NLB → EC2 TCPアプリ

```text
Client
  → NLB
  → EC2 TCP service
```

向く要件。

```text
HTTPではないTCPアプリ
高スループット/低レイテンシ
固定IPが必要
クライアントIPを保持したい
```

何が良いか。

```text
L4で高速に分散できる
固定IPやElastic IPを使いやすい
HTTP以外のプロトコルを扱える
```

---

### 5.2 NLB → ECS TCP/UDP Service

```text
Client
  → NLB
  → ECS tasks running TCP/UDP service
```

向く要件。

```text
コンテナで非HTTPプロトコルを提供する
ゲームサーバーや独自TCPサービス
UDPを扱う
```

何が良いか。

```text
HTTPではないコンテナサービスを公開できる
高性能L4分散ができる
```

---

### 5.3 NLB → ALB

```text
Client
  → NLB
  → ALB
  → ECS / EC2
```

向く要件。

```text
固定IPやPrivateLink入口が必要
同時にHTTPのpath/host routingも必要
```

何が良いか。

```text
NLBでL4入口や固定IP/PrivateLink要件を満たす
ALBでL7ルーティングを行う
```

注意。

```text
単純なHTTP/HTTPSだけならALB単体でよい。
NLB→ALBは、固定IPやPrivateLinkなど明確なL4入口要件があるときに検討する。
```

---

### 5.4 PrivateLink Consumer → Interface Endpoint → NLB → Provider Service

```text
Consumer VPC
  Client
    → Interface Endpoint
    → PrivateLink

Provider VPC
  NLB
    → Service on ECS / EC2 / IP targets
```

向く要件。

```text
SaaSを顧客VPCへprivate公開したい
共有サービスを複数VPCからprivateに使わせたい
CIDR重複がある
VPC全体の到達性を与えたくない
```

何が良いか。

```text
サービス単位で公開できる
Consumer側はprivate IPで接続できる
VPC Peering/TGWのような広い到達性を与えない
CIDR重複に強い
```

これはSAP-C02頻出。

```text
PrivateLinkのProvider側 = NLB + Endpoint Service
Consumer側 = Interface Endpoint
```

---

### 5.5 Global Accelerator → NLB → TCP/UDP Service

```text
Global users
  → Global Accelerator Anycast IP
  → NLB
  → TCP/UDP service
```

向く要件。

```text
グローバル固定IPが必要
TCP/UDP通信を高速化したい
複数リージョンへfailoverしたい
CloudFrontでは扱いにくい非HTTP通信
```

何が良いか。

```text
Anycast IPで入口を安定化できる
AWS global networkへ早く乗せられる
TCP/UDPワークロードに使える
```

CloudFrontとの違い。

```text
CloudFront = HTTP/HTTPS CDN/cache
Global Accelerator = TCP/UDP入口最適化
```

---

## 6. ALB/NLBが「後段」に見えるパターン

厳密にはALB/NLBは通常、アプリの前段に置く。ただし、CloudFrontやGlobal Acceleratorなど、さらに外側の入口があると、相対的に後段に見える。

### 6.1 CloudFrontの後段にALB

```text
User
  → CloudFront
  → ALB
  → Application
```

この場合、ALBはユーザーから見れば直接入口ではなく、CloudFrontのorigin。

何が良いか。

```text
CloudFrontでedge/cache/WAF
ALBでアプリルーティング/health check
役割を分離できる
```

---

### 6.2 Global Acceleratorの後段にALB/NLB

```text
User
  → Global Accelerator
  → ALB / NLB
  → Application
```

何が良いか。

```text
GAでAnycast IPとグローバル経路最適化
ALBでHTTP L7 routing
NLBでTCP/UDP L4 routing
```

---

### 6.3 API Gatewayの後段にNLB/VPC Link

```text
Client
  → API Gateway
  → VPC Link
  → NLB
  → private service
```

向く要件。

```text
API Gatewayをpublic API入口にしたい
backendはVPC内private serviceにしたい
private integrationが必要
```

何が良いか。

```text
外部にはAPI Gatewayだけを見せる
backend serviceをprivate subnet内に隠せる
API管理とVPC内部サービスを接続できる
```

注意。

```text
API GatewayはAPI管理の入口。
NLBはVPC内部のprivate serviceへの接続点として使われる。
```

---

## 7. どのサービスの前にALB/NLBが来るのか

| 後ろのサービス | ALBが自然なケース | NLBが自然なケース |
|---|---|---|
| EC2 | HTTP/HTTPS Webアプリ | TCP/UDP/固定IP/高性能 |
| ECS | Web/APIコンテナ、path routing | TCP/UDPコンテナ、PrivateLink提供 |
| EKS | Ingress, HTTP/gRPC | TCP/UDP Service, NLB type service |
| Lambda | HTTP pathの一部をLambdaへ | 通常はNLBではなくALB/API Gatewayを検討 |
| API Gateway | 通常ALBは前に置かない。CloudFrontは前段に来る | VPC Linkの後段にNLBが来ることがある |
| CloudFront | CloudFrontのoriginとしてALB | CloudFrontのoriginとしてNLBは通常主役ではない |
| PrivateLink | ALB単体ではなく、Provider側はNLBが頻出 | NLB + Endpoint Service |
| Transfer Family | 通常不要。Transfer Family自体がSFTP入口 | 自前SFTPならNLBだが、管理負荷最小ならTransfer Family |
| IoT Core | 通常不要。IoT Core自体が入口 | 通常不要 |
| S3 | ALB/NLBの後ろに直接置くものではない | S3には直接置かない。CloudFrontやTransfer Familyを使う |

---

## 8. ユースケース別の具体例

### 8.1 Webアプリを公開したい

```text
User
  → CloudFront(optional)
  → ALB
  → ECS/EC2
```

選ぶ理由。

```text
HTTP/HTTPSだからALB
global/cache/WAFを強めたいならCloudFrontも前段に置く
```

---

### 8.2 複数マイクロサービスをURLで分けたい

```text
User
  → ALB
      /users/*  → user-service
      /orders/* → order-service
      /admin/*  → admin-service
```

選ぶ理由。

```text
path-based routingはALBの得意分野
NLBではURL pathを見ない
```

---

### 8.3 gRPCサービスをECS/EKSへ振り分けたい

```text
Client service
  → ALB
  → gRPC services on ECS/EKS
```

選ぶ理由。

```text
gRPC/HTTP2のL7負荷分散はALBで考える
```

---

### 8.4 SFTPでファイルを取引先に提供したい

```text
Partner SFTP Client
  → AWS Transfer Family
  → S3
```

選ぶ理由。

```text
既存SFTPクライアントを変えない
マネージドSFTP入口を使う
ALB/NLBで自作SFTPを公開しない
```

ただし、自前SFTPをどうしても運用するなら、SFTPはHTTPではなくTCPなので、ALBではなくNLB寄りになる。

---

### 8.5 SaaSを顧客VPCにprivate公開したい

```text
Consumer VPC
  Interface Endpoint
    → PrivateLink
Provider VPC
  NLB
    → SaaS service
```

選ぶ理由。

```text
PrivateLink Provider側はNLB + Endpoint Serviceが頻出
VPC全体をつながずサービス単位で公開できる
```

---

### 8.6 VPC内のprivate APIをAPI Gatewayから呼びたい

```text
Client
  → API Gateway
  → VPC Link
  → NLB
  → private ECS/EC2 service
```

選ぶ理由。

```text
API Gatewayを外部入口にしてAPI管理する
backendはprivate VPC内に置く
VPC LinkでNLB経由接続する
```

---

### 8.7 TCP/UDPゲームサーバーを公開したい

```text
Game client
  → Global Accelerator(optional)
  → NLB
  → Game servers on EC2/ECS
```

選ぶ理由。

```text
TCP/UDPはNLB
グローバル低遅延/固定Anycast IPならGlobal Accelerator
```

---

## 9. ALB/NLBを置くことで何が嬉しいか

### ALBを置くメリット

```text
HTTP/HTTPSの入口を統一できる
Path/Host/Headerで振り分けられる
Targetのhealth checkができる
TLS終端を集約できる
ECS/EKS/EC2/Lambdaと連携できる
WAFやCognito/OIDC認証と組み合わせやすい
blue/greenやcanary的な切り替えに使いやすい
```

### NLBを置くメリット

```text
TCP/TLS/UDPの入口を統一できる
低レイテンシ/高スループット
固定IP/Elastic IPを使いやすい
非HTTPプロトコルに対応できる
PrivateLinkのEndpoint Service前段にできる
クライアントIPを保持しやすい
```

---

## 10. ALB/NLBを置かない方がよい場合

| 要件 | 避ける理由 | 代替候補 |
|---|---|---|
| S3静的配信 | S3の前にALB/NLBは置かない | CloudFront + S3 |
| SFTPをマネージド提供 | 自作LB配下SFTPは運用負荷が高い | Transfer Family |
| 単純なLambda HTTPS公開 | ALB/API Gatewayが過剰な場合がある | Lambda Function URL |
| 本格API管理 | ALBだけではAPI管理が弱い | API Gateway |
| IoT MQTT通信 | ALB/NLBではIoT機能がない | IoT Core |
| VPC全体接続 | ALB/NLBではネットワーク接続にならない | TGW / VPC Peering / VPN / DX |

---

## 11. SAP-C02判断フロー

```text
通信はHTTP/HTTPS？
  → ALB / API Gateway / CloudFront を検討

URL path/host/headerで振り分ける？
  → ALB

API認証、スロットリング、Usage Plan？
  → API Gateway

CDN/cache/edge WAF/global配信？
  → CloudFront

TCP/UDP/TLSなど非HTTP？
  → NLB

固定IPが必要？
  → NLB or Global Accelerator

PrivateLinkでサービス公開？
  → NLB + Endpoint Service

gRPC？
  → ALB

SFTP/FTPS/FTP/AS2？
  → Transfer Family

IoT MQTT/Device Shadow/IoT Rules？
  → IoT Core
```

---

## 12. よくある誤答パターン

| 誤答 | なぜ危険か |
|---|---|
| SFTPをALB配下で公開する | SFTPはHTTPではない。さらに管理負荷最小ならTransfer Family |
| path-based routingをNLBでやる | NLBはL4。URL pathは見ない |
| 固定IP要件でALBを選ぶ | NLB/Global Acceleratorを検討 |
| API Gatewayの代わりにALBでAPI管理する | ALBはLB。API key/usage plan等はAPI Gatewayが得意 |
| CloudFrontがあるからALB不要と考える | CloudFrontはCDN、ALBはアプリ分散 |
| NLBがあるからPrivateLinkだと考える | PrivateLinkにはEndpoint Service/Interface Endpointが必要 |
| ALB/NLBをS3の前段に直接置く | S3配信はCloudFront/S3、SFTPならTransfer Family |
| IoT通信をALB/API Gatewayだけで処理する | MQTT/Device Shadow/IoT RulesならIoT Core |

---

## 13. 最短暗記

```text
ALB = HTTPを理解する入口
NLB = TCP/UDPを速く流す入口
CloudFront = HTTPを世界中に近づける入口
API Gateway = APIとして管理する入口
IoT Core = デバイス通信の入口
Transfer Family = SFTP等のファイル転送入口
Global Accelerator = TCP/UDPをグローバル最適化する入口
```

```text
サービスの前にLBを置く理由:
  入口を1つにする
  正常なターゲットへ流す
  スケールアウトを隠す
  TLS/証明書を集約する
  ルーティングを集約する
  障害ターゲットを切り離す
```

---

## 関連ページ

- [HTTP / HTTPS / WebSocket / MQTT / gRPC / TCP / UDP とAWS入口サービス](protocols-and-aws-entrypoints.md)
- [Network / Routing / Security Boundary 混同整理](network-routing-security-confusions.md)
- [AWS Endpoint / ENI / VIF 完全整理](endpoints-eni-vif.md)
- [AWS PrivateLink / VPC Endpoints](../services/networking/privatelink.md)
- [Elastic Load Balancing](../services/networking/elb.md)
- [Amazon CloudFront](../services/networking/cloudfront.md)
- [AWS Global Accelerator](../services/networking/global-accelerator.md)
- [AWS Transfer Family](../services/migration/transfer-family.md)
