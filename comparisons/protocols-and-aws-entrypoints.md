# HTTP / HTTPS / WebSocket / MQTT / gRPC / TCP / UDP とAWS入口サービス

## 何のためのページか

AWS設計では、通信方式とAWSサービスの入口を混同しやすい。

例えば、CloudFront、API Gateway、ALB、NLB、AWS IoT Core は、いずれも外部からの通信を受ける入口になり得るが、役割は大きく違う。

このページでは、HTTP / HTTPS / REST API / WebSocket / MQTT / gRPC / TCP / UDP が、どのAWSサービスを通りやすいかをSAP-C02向けに整理する。

---

## まず結論

```text
CloudFront = CDN / Edge / Cache / Origin protection
API Gateway = API管理の入口
ALB = HTTP/HTTPS/gRPCのL7ロードバランサー
NLB = TCP/TLS/UDPのL4ロードバランサー
IoT Core = IoTデバイス通信 / MQTT / ルール処理
Global Accelerator = TCP/UDPのグローバル入口最適化
```

---

## 1. ProtocolとAPIスタイルを分ける

まず、すべてを同じ「データの種類」として扱わない。

| 用語 | 種類 | 一言 |
|---|---|---|
| HTTP | アプリケーション層プロトコル | Web/API通信の基本 |
| HTTPS | HTTP + TLS | 暗号化されたHTTP |
| REST | API設計スタイル | HTTP上でGET/POST/PUT/DELETE等を使う設計 |
| HTTP API | API GatewayのAPI種別 / HTTPベースAPI | RESTより軽量なAPI Gatewayの選択肢 |
| WebSocket | 双方向通信プロトコル | 接続を維持して双方向通信 |
| MQTT | 軽量Pub/Subプロトコル | IoTデバイス通信に強い |
| gRPC | HTTP/2ベースのRPC | マイクロサービス間通信に強い |
| TCP | トランスポート層 | 信頼性のある接続型通信 |
| UDP | トランスポート層 | 低遅延・軽量通信 |

注意点。

```text
RESTはプロトコルではなくAPI設計スタイル。
HTTPSは通信プロトコル。
API Gatewayはサービス。
CloudFrontはCDN。
IoT CoreはIoTメッセージブローカー/デバイス通信基盤。
```

---

## 2. HTTP / HTTPS

HTTP/HTTPSは最も広い入口を持つ。

```text
Client
  → HTTP/HTTPS
  → CloudFront / API Gateway / ALB / Lambda Function URL / IoT Core
```

| 入口サービス | 向く用途 | 試験での読み方 |
|---|---|---|
| CloudFront | CDN、キャッシュ、WAF、オリジン保護、グローバル配信 | edge / cache / static / global / WAF |
| API Gateway | API公開、認証、スロットリング、Lambda連携 | REST API / HTTP API / API管理 |
| ALB | HTTP/HTTPSのL7負荷分散 | ECS/EC2/Lambdaへのアプリ分散 |
| Lambda Function URL | Lambdaの簡易HTTPS公開 | 簡易・低機能・小規模 |
| IoT Core HTTPS | IoTデバイスからのメッセージ送信 | デバイス / センサー / 証明書 / IoT Rules |

### CloudFrontの注意

CloudFrontは「一方向通信サービス」ではない。

```text
Client
  → CloudFront
  → Origin
  ← CloudFront
  ← Client
```

HTTP/HTTPSのrequest/responseとしては往復する。POSTなどの動的リクエストをオリジンへ転送することもできる。

ただし、CloudFrontはIoTデータ受信処理、デバイス管理、MQTT broker、Device Shadow、IoT Rulesのためのサービスではない。

```text
CloudFront = request/responseのエッジ中継・キャッシュ・保護
IoT Core   = IoTデバイス通信管理・Pub/Sub・ルール処理
```

---

## 3. REST API / HTTP API

REST APIはHTTP上のAPI設計スタイル。AWSではAPI GatewayやALBで扱うことが多い。

```text
Client / Mobile App / External System
  → HTTPS REST/HTTP request
  → API Gateway
  → Lambda / ECS / HTTP backend / AWS service integration
```

### API Gatewayが向く要件

- REST API / HTTP APIを公開したい
- 認証/認可をAPI単位で扱いたい
- スロットリング、API Key、Usage Planを使いたい
- LambdaやAWSサービス統合を使いたい
- WebSocket APIをマネージドに扱いたい

### ALBが向く要件

- ECS/EC2上のWebアプリへHTTP/HTTPSで負荷分散したい
- パスベース/ホストベースルーティングをしたい
- gRPCやHTTP/2を使うアプリケーションを分散したい
- API管理よりロードバランシングが主目的

---

## 4. WebSocket

WebSocketは、HTTP Upgradeから始まる持続的な双方向通信。

```text
Client
  ⇄ WebSocket
  ⇄ API Gateway WebSocket API
  ⇄ Lambda / Backend
```

向くもの。

```text
チャット
リアルタイム通知
リアルタイムダッシュボード
双方向アプリ
```

IoT文脈では、MQTT over WebSocketも出る。

```text
Device / Browser client
  ⇄ MQTT over WebSocket
  ⇄ AWS IoT Core
```

### CloudFrontとWebSocket

CloudFrontはWebSocket通信を中継できる。ただし、CloudFrontの主目的はCDN/edge配信/オリジン保護であり、双方向アプリの接続管理そのものはAPI Gateway WebSocket APIやIoT Coreで考える。

---

## 5. MQTT

MQTTは、軽量なPub/Sub型プロトコル。IoTデバイス通信で頻出。

```text
Device
  → publish topic
  → AWS IoT Core message broker
  → IoT Rule
  → Lambda / Kinesis / S3 / DynamoDB / Timestream
```

向くもの。

```text
センサー
車両
工場機器
スマートホーム
大量デバイス
低帯域環境
証明書ベースのデバイス認証
```

API Gatewayではなく、まずAWS IoT Coreを見る。

---

## 6. AWS IoT Core HTTPS

AWS IoT CoreはMQTTだけでなく、HTTPSでもデバイスからメッセージを受けられる。

```text
Device
  → HTTPS publish
  → AWS IoT Core
  → IoT Rule
  → Lambda / Kinesis / S3 / DynamoDB
```

ただし、IoT CoreのHTTPSは汎用Web APIというより、IoTデバイスのメッセージ送信口として考える。

```text
HTTPSを受けられるか？
  → API GatewayもIoT Coreも可能

何に向いているか？
  → 汎用APIならAPI Gateway
  → IoTデバイス通信ならIoT Core
```

---

## 7. gRPC

gRPCはHTTP/2ベースの高性能RPC。SAP-C02では、まずALBと結びつけて考えるのが安全。

```text
Client / Service
  → gRPC
  → ALB
  → ECS / EKS / EC2
```

向くもの。

```text
マイクロサービス間通信
低レイテンシRPC
HTTP/2ベース通信
サービス間API
```

注意点。

```text
API GatewayはREST API / HTTP API / WebSocket APIの文脈で整理する。
gRPCの入口としてはALBを第一候補にする。
```

---

## 8. TCP / UDP

TCP/UDPのL4通信では、NLBやGlobal Acceleratorが候補になる。

```text
Client
  → TCP / TLS / UDP
  → NLB
  → EC2 / ECS / Appliance
```

| 通信 | 入口 | 向く用途 |
|---|---|---|
| TCP | NLB | 高性能L4通信 |
| TLS | NLB | TLS終端/パススルー系 |
| UDP | NLB | UDPワークロード |
| TCP/UDPのグローバル入口 | Global Accelerator | Anycast IP、リージョンフェイルオーバー、低遅延化 |

向くもの。

```text
ゲーム
VoIP
独自プロトコル
DNS系ワークロード
高性能L4通信
固定IPが必要な入口
```

---

## 9. サービス別の役割整理

| サービス | レイヤー | 主な通信 | 役割 |
|---|---|---|---|
| CloudFront | Edge/CDN | HTTP/HTTPS, WebSocket中継 | キャッシュ、配信、WAF、オリジン保護 |
| API Gateway | API管理 | HTTPS, WebSocket | API公開、認証、スロットリング |
| ALB | L7 LB | HTTP/HTTPS/gRPC | アプリ負荷分散 |
| NLB | L4 LB | TCP/TLS/UDP | 高性能L4負荷分散 |
| IoT Core | IoT Message Broker | MQTT, MQTT over WebSocket, HTTPS | デバイス通信、Pub/Sub、IoT Rules |
| Global Accelerator | Global network edge | TCP/UDP | Anycast IP、グローバル高速化、リージョン切替 |
| Lambda Function URL | Function endpoint | HTTPS | Lambda簡易公開 |

---

## 10. シナリオ別の選択

| シナリオ | 選ぶ候補 | 理由 |
|---|---|---|
| 静的Webサイトをグローバル配信 | CloudFront + S3 | CDN/キャッシュ/HTTPS/WAF |
| S3 private originを保護 | CloudFront + OAC | S3直接アクセスを防ぐ |
| REST APIを公開 | API Gateway | API管理/認証/スロットリング |
| ECS/EC2のWebアプリへL7分散 | ALB | HTTP/HTTPS負荷分散 |
| gRPCサービスを分散 | ALB | gRPC/HTTP2対応のL7分散 |
| TCP/UDPアプリを分散 | NLB | L4高性能 |
| グローバル固定IP/低遅延化 | Global Accelerator | Anycast IP/TCP/UDP高速化 |
| センサーが少量HTTPS送信 | API Gateway + Lambda + DynamoDB | シンプルな汎用APIなら十分 |
| 大量IoTデバイス、MQTT、証明書 | IoT Core | デバイス通信管理/PubSub/Rules |
| リアルタイム双方向Webアプリ | API Gateway WebSocket API | WebSocket接続管理 |
| IoTの双方向/PubSub | IoT Core MQTT / MQTT over WSS | IoTメッセージブローカー |

---

## 11. CloudFrontをデータ受信に使う場合の注意

CloudFront経由でPOSTをオリジンへ流すことはできる。

```text
Sensor / Client
  → HTTPS POST
  → CloudFront
  → API Gateway / ALB / Custom Origin
  → Backend
```

しかし、次の要件があるならCloudFront単体ではなく、API GatewayやIoT Coreを見る。

```text
API認証/認可
リクエスト単位のスロットリング
Lambda連携
デバイス証明書
MQTT
Device Shadow
IoT Rules
大量デバイス接続管理
```

覚え方。

```text
CloudFrontは入口を速く・近く・守る。
API GatewayはAPIとして受ける。
IoT Coreはデバイス通信として受ける。
```

---

## 12. SAP-C02判断フロー

```text
静的/動的コンテンツをグローバル高速配信？
  → CloudFront

Private S3 originを直接アクセスから守る？
  → CloudFront + OAC

汎用HTTPS APIを公開？
  → API Gateway

WebSocketでリアルタイムアプリ？
  → API Gateway WebSocket API

HTTP/HTTPSアプリをEC2/ECSへ振り分け？
  → ALB

gRPCアプリを負荷分散？
  → ALB

TCP/UDP高性能入口？
  → NLB

TCP/UDPのグローバル低遅延/固定Anycast IP？
  → Global Accelerator

IoTデバイス、MQTT、X.509証明書、Device Shadow？
  → AWS IoT Core

センサーが単純にHTTPSで少量データ送信？
  → API Gateway + Lambda + DynamoDB も候補
```

---

## 13. よくある誤答パターン

| 誤答 | なぜ危険か |
|---|---|
| CloudFrontは一方向通信だからPOSTやAPI前段に使えない | request/responseやPOST転送は可能。ただしIoT処理基盤ではない |
| CloudFrontをIoT Core代わりにする | デバイス管理、MQTT、IoT Rules、Device Shadowがない |
| HTTPSを受けるならIoT CoreとAPI Gatewayは同じ | API Gatewayは汎用API、IoT Coreはデバイス通信基盤 |
| MQTTをAPI Gatewayで受ける | MQTTはIoT Coreで考える |
| WebSocketなら必ずIoT Core | アプリ双方向ならAPI Gateway WebSocket APIが自然 |
| gRPCならAPI Gateway | SAP-C02ではALBを第一候補にする |
| ALBとAPI Gatewayを同じAPI入口として扱う | ALBはL7負荷分散、API GatewayはAPI管理 |
| NLBでHTTPのパスベースルーティングをする | パス/ホストベースはALB |
| Global AcceleratorをCDNとして使う | GAはTCP/UDP入口最適化。キャッシュはCloudFront |
| Lambda Function URLを本格API管理に使う | 簡易公開向け。API管理はAPI Gateway |

---

## 14. 最短暗記

```text
CloudFront = CDN / Edge / Cache / WAF / Origin protection
API Gateway = REST/HTTP/WebSocket API管理
ALB = HTTP/HTTPS/gRPC L7負荷分散
NLB = TCP/TLS/UDP L4負荷分散
IoT Core = MQTT/HTTPS/WebSocketによるIoTデバイス通信
Global Accelerator = TCP/UDPのグローバル入口最適化
```

---

## 関連ページ

- [AWS Endpoint / ENI / VIF 完全整理](endpoints-eni-vif.md)
- [Endpoint / ENI / VIF / PrivateLink 理解補完レジュメ](endpoints-eni-vif-remedial-resume.md)
- [Networking Foundations Deep Dive](networking-foundations-deep-dive.md)
- [Networking Connectivity Options](networking-options.md)
- [Amazon CloudFront](../services/networking/cloudfront.md)
- [Elastic Load Balancing](../services/networking/elb.md)
- [AWS Global Accelerator](../services/networking/global-accelerator.md)
