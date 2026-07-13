# AWS「説明の説明」

> AWSの説明を読んでも分からない原因は、AWSサービスを知らないことだけではない。説明文の中に隠れている「誰が」「何を」「どこへ」「なぜ」「どの条件で」を展開できていないことが多い。

このページは、個別サービスの説明ではなく、**AWSの説明文そのものを読み解くための基礎読本**である。

たとえば、次の一文を考える。

> SQSはプロデューサーとコンシューマーを疎結合にする。

この文だけでは、初学者には次の疑問が残る。

- プロデューサーとは誰か
- コンシューマーとは誰か
- 何が結合していたのか
- SQSは間に入って何を保存するのか
- 相手が停止したとき何が変わるのか
- なぜ直接呼び出すより障害に強くなるのか
- 代わりに何を失うのか

「説明の説明」とは、このような圧縮された一文を、通信・データ・時間・責任・トレードオフへ展開することである。

---

## 目次

1. [なぜAWSの説明は分かりにくいのか](#1-なぜawsの説明は分かりにくいのか)
2. [すべての説明を七つの質問へ分解する](#2-すべての説明を七つの質問へ分解する)
3. [説明文の基本構造](#3-説明文の基本構造)
4. [AWS説明で使われる動詞を理解する](#4-aws説明で使われる動詞を理解する)
5. [位置関係を表す言葉](#5-位置関係を表す言葉)
6. [レイヤーとプレーンを分ける](#6-レイヤーとプレーンを分ける)
7. [パブリックとプライベート](#7-パブリックとプライベート)
8. [同期と非同期](#8-同期と非同期)
9. [ステートレスとステートフル](#9-ステートレスとステートフル)
10. [マネージドとサーバーレス](#10-マネージドとサーバーレス)
11. [可用性・冗長化・複製・バックアップ・DR](#11-可用性冗長化複製バックアップdr)
12. [性能を表す言葉](#12-性能を表す言葉)
13. [認証・認可・暗号化](#13-認証認可暗号化)
14. [再試行・冪等性・タイムアウト](#14-再試行冪等性タイムアウト)
15. [AWSの代表的な一文を展開する](#15-awsの代表的な一文を展開する)
16. [良い説明を作るテンプレート](#16-良い説明を作るテンプレート)
17. [分かりにくい説明の典型](#17-分かりにくい説明の典型)
18. [説明を読んだときの確認問題](#18-説明を読んだときの確認問題)
19. [このリポジトリでの使い方](#19-このリポジトリでの使い方)

---

# 1. なぜAWSの説明は分かりにくいのか

## 1.1 一文の中に前提知識が隠れている

AWSの資料では、次のような一般IT用語が説明なしに使われる。

- endpoint
- proxy
- gateway
- route
- session
- credential
- replica
- cache
- queue
- failover
- throughput
- connection

AWS固有のサービス名を覚えても、これらの一般語の意味が曖昧なら、説明全体が曖昧になる。

## 1.2 サービス名が主語になり、本当の動作主体が省略される

> CloudFrontがコンテンツをキャッシュする。

この文では、実際には次の主体が存在する。

```text
Viewer
  → CloudFront edge location
      → cache lookup
          ├─ hit  → cached response
          └─ miss → origin request
                       → responseをedgeへ保存
                       → viewerへ返す
```

「CloudFront」という一語の中に、エッジロケーション、キャッシュキー、オリジン、TTL、リクエストとレスポンスが隠れている。

## 1.3 目的と仕組みが混ざっている

> Multi-AZは可用性を高める。

これは目的の説明であり、仕組みの説明ではない。

仕組みまで展開すると、次のようになる。

```text
Primary DB
  → 別AZのStandbyへ複製
  → Primary障害を検知
  → Standbyへフェイルオーバー
  → 接続先を切り替える
```

さらに、「読み取り性能を上げる仕組みではない」という不採用理由まで必要になる。

## 1.4 正しい説明でも、比較対象がないと判断できない

> PrivateLinkはプライベート接続を提供する。

この説明だけでは、VPC PeeringやTransit Gatewayとの違いが分からない。

- Peering：VPC同士のIP通信経路を作る
- Transit Gateway：多数ネットワークを中継する
- PrivateLink：提供対象のサービスだけを利用側VPCへ見せる

説明は、**何であるか**だけでなく、**何ではないか**を含めて初めて設計判断に使える。

---

# 2. すべての説明を七つの質問へ分解する

AWSサービスの説明を読んだら、次の七つを埋める。

| 質問 | 確認すること |
|---|---|
| 誰が | client、user、service、administrator、producer、consumer |
| 何を | request、event、message、packet、object、record、credential |
| どこからどこへ | source、destination、前段、後段、別AZ、別Region |
| いつ | request時、event発生時、障害時、定刻、非同期、期限切れ時 |
| どうやって | route、proxy、queue、replication、API、DNS、role assumption |
| なぜ | 可用性、性能、運用削減、セキュリティ、コスト、疎結合 |
| 代わりに何を失うか | 遅延、複雑性、整合性、コスト、制御、即時性 |

## 例：SQSは疎結合にする

| 質問 | 展開した答え |
|---|---|
| 誰が | 注文APIと注文処理ワーカー |
| 何を | 注文処理要求を表すメッセージ |
| どこからどこへ | API → SQS Queue → Worker |
| いつ | APIが注文を受けた後、Workerが取得可能になった時 |
| どうやって | Queueがメッセージを保持し、Workerがpollして処理する |
| なぜ | Worker停止中でもAPIと処理要求を分離できる |
| 代償 | 即時完了ではない、重複処理や順序を設計する必要がある |

## 例：SCPは権限のガードレールである

| 質問 | 展開した答え |
|---|---|
| 誰が | Organizations配下アカウントのIAM principal |
| 何を | 実行可能なAWS API actionの最大範囲 |
| どこへ | RootまたはOU、Accountに適用 |
| いつ | AWSがアクセス許可を評価するとき |
| どうやって | IAMでAllowされても、SCPの範囲外なら実行不可にする |
| なぜ | アカウント管理者でも越えられない組織統制を作る |
| 代償 | 設計を誤ると必要操作まで組織単位で止める |

---

# 3. 説明文の基本構造

説明文は、次の形へ書き換える。

```text
主体
  が
入力
  を受け取り
処理
  を行い
出力 / 状態変化
  を生み
目的
  を達成する。
ただし
条件 / 制約 / 代償
  がある。
```

## 3.1 短い説明

> ALBはトラフィックを複数ターゲットへ分散する。

## 3.2 展開した説明

```text
ClientがHTTP/HTTPS requestをALBのDNS名へ送る。
ALBのlistenerがprotocolとportを受け付ける。
Listener ruleがhost、path、headerなどを評価する。
Ruleがtarget groupを選ぶ。
ALBはhealthyなEC2、ECS task、IP、Lambdaなどへrequestを転送する。
複数AZのhealthy targetへ処理を分散することで、負荷分散と可用性を高める。
ただし、ALB自体がアプリケーションコードを実行するわけではない。
```

この展開で重要なのは、名詞を増やすことではない。**処理の順番を見えるようにすること**である。

---

# 4. AWS説明で使われる動詞を理解する

## 4.1 resolve：名前を接続先情報へ変換する

例：Route 53が名前解決する。

```text
app.example.com
  → DNS query
  → DNS record / routing policy
  → IP address or AWS resource alias
```

名前解決は通信そのものではない。通信前に「どこへ接続するか」を得る処理である。

## 4.2 route：宛先に応じて次の経路を選ぶ

例：VPC route tableがpacketの次の送信先を決める。

```text
Destination IP
  → longest prefix match
  → local / IGW / NAT Gateway / TGW / VGW / Endpoint
```

Route tableはアプリケーションのURL pathを見ない。IP宛先を基に経路を選ぶ。

## 4.3 forward：受け取ったものを後段へ渡す

例：ALBがrequestをtargetへforwardする。

Forwardは複製とは限らない。受信した通信やメッセージを別の処理主体へ渡す。

## 4.4 proxy：代理として通信を受け、別の相手へ接続する

```text
Client
  → Proxy
      → Backend
```

Clientから見るとProxyが相手に見える。Backendから見るとProxyが接続元に見える場合がある。

- Forward proxy：Client側の代理
- Reverse proxy：Server側の代理
- RDS Proxy：ApplicationとDB connectionの間の代理

## 4.5 terminate：その場所で通信や暗号処理を終端する

例：ALBでTLS終端する。

```text
Client ==TLS==> ALB
ALB  ----HTTP or TLS----> Target
```

TLS終端は「通信がそこで終了する」という意味ではない。**そのTLS sessionをALBが復号し、後段へ別の接続を作る**という意味である。

## 4.6 distribute：複数の処理先へ割り振る

Load Balancerはrequestやconnectionを複数targetへ割り振る。必ずしも一つのrequestを複数へ複製するわけではない。

- Load balancing：一件をいずれかのtargetへ
- Fan-out：一件を複数consumerへ

この二つを混同しない。

## 4.7 replicate：同じデータの複製を別の場所へ作る

目的は複数ある。

- 可用性
- 読み取り性能
- DR
- 地理的低遅延

同じ「replica」でも、用途と整合性が違う。

## 4.8 backup：過去時点へ戻せるコピーを保持する

Replicaは現在の変更を追随する。Backupは過去時点を保持する。

```text
誤削除
  → Replicaにも反映される可能性
  → Backup / PITRから過去時点へ復元
```

## 4.9 cache：再利用する値を近い場所へ一時保存する

Cacheは正本とは限らない。

- CloudFront：HTTP responseをedgeに保存
- ElastiCache：application dataをmemoryに保存
- DAX：DynamoDB read結果をcache
- DNS cache：名前解決結果をTTL期間保存

何をcacheし、正本がどこにあるかを確認する。

## 4.10 buffer：速度差を吸収するため一時的に貯める

SQSはproducerの送信速度とconsumerの処理速度の差を吸収する。

```text
Producer 100件/秒
  → Queueへ蓄積
Consumer 20件/秒
  → 時間をかけて処理
```

Bufferは処理能力を直接増やさない。溢れるまでの時間と再処理性を作る。

## 4.11 orchestrate：複数処理の順序と状態を管理する

Step Functionsは、単にeventを転送するのではなく、処理の進行状態を持つ。

```text
Validate
  → Charge
  → Reserve inventory
  → Ship
  → failure時はcompensation
```

## 4.12 trigger：条件成立をきっかけに処理を開始する

Triggerは、必ずHTTP requestを意味しない。

- EventBridge schedule
- S3 event
- DynamoDB Streams
- CloudWatch Alarm
- API request

「何がきっかけか」を確認する。

## 4.13 scale：処理能力を需要へ合わせる

- Scale up：一台を大きくする
- Scale out：台数を増やす
- Scale in：台数を減らす
- Scale down：一台を小さくする

Auto Scalingは、何を、どの指標で、どの速さで増減させるかまで設計する。

## 4.14 authenticate：相手が誰か確認する

例：Cognito User Poolがloginを検証しJWTを発行する。

## 4.15 authorize：その相手が何をしてよいか判断する

例：IAMがprincipal、action、resource、conditionを評価する。

認証済みでも、すべての操作が許可されるわけではない。

## 4.16 encrypt：読める情報を鍵なしでは読めない形にする

- at rest：保存時
- in transit：通信時
- client-side：送信前にclientで暗号化
- server-side：serviceが保存時に暗号化

暗号化の説明では「誰が鍵を持つか」「どこで復号するか」を確認する。

## 4.17 observe：状態を測定・記録する

- Metrics：数値の時系列
- Logs：出来事の記録
- Traces：一つのrequestが複数serviceを通る経路
- Events：状態変化の通知

CloudWatchという名前だけで、何を観測しているかを省略しない。

## 4.18 fail over：障害時に待機系へ役割を移す

```text
Primary failure
  → health detection
  → traffic / role切替
  → Standby becomes active
```

Failoverには、検知時間、切替時間、接続再確立、データ整合性が関係する。

## 4.19 recover：壊れた状態から業務可能な状態へ戻す

Recoveryは、単にresourceを起動することではない。

- dataを戻す
- applicationを起動する
- networkを切り替える
- secretやcertificateを利用可能にする
- 業務確認を行う

## 4.20 provision：必要なresourceを作成・割り当てる

Provisioningは、software deploymentと同じではない。

```text
Provision infrastructure
  → VPC / EC2 / RDSを作る
Deploy application
  → code / container image / configurationを配置する
```

---

# 5. 位置関係を表す言葉

## 5.1 前段と後段

```text
User → CloudFront → ALB → ECS → RDS
```

Userから見て先に通るものが前段、後で通るものが後段である。

ただし「前段」は絶対位置ではない。

- ALBから見ればCloudFrontは前段
- ECSから見ればALBは前段
- ALBから見ればECSは後段

何を基準にした前後かを確認する。

## 5.2 sourceとdestination

通信の送信元と宛先。

Security Groupのinbound ruleでは、resourceへ入ってくる通信のsourceを見る。Outboundではdestinationを見る。

## 5.3 clientとserver

- Client：処理を依頼する側
- Server：依頼を受けて応答する側

同じserviceでも、場面により両方になる。

```text
Browser(client) → API(server)
API(client)     → RDS(server)
```

## 5.4 producerとconsumer

- Producer：messageやeventを作る
- Consumer：messageやeventを受けて処理する

Request/responseのclient/serverとは異なり、非同期処理で使われることが多い。

## 5.5 originとedge

- Origin：元データや元responseを持つ場所
- Edge：利用者に近い配信地点

CloudFrontではS3、ALB、EC2などがoriginになり、edge cacheからviewerへ返す。

## 5.6 ingressとegress

- Ingress：外から中へ入る通信
- Egress：中から外へ出る通信

Kubernetes Ingressは単なる「入ってくる通信」という一般語より狭く、HTTP(S)入口を定義するresourceを指す場合がある。

## 5.7 north-southとeast-west

- North-south：外部と内部の通信
- East-west：内部service間の通信

WAF、CloudFront、ALBはnorth-southで語られやすい。Service Meshや内部ロードバランサーはeast-westで語られやすい。

---

# 6. レイヤーとプレーンを分ける

## 6.1 ネットワークレイヤー

| 層 | 主に見るもの | 代表例 |
|---|---|---|
| L3 | IP address、route | VPC route、Transit Gateway |
| L4 | TCP/UDP、port、connection | NLB、Security Group |
| L7 | HTTP method、host、path、header | ALB、API Gateway、WAF |

L7 serviceはHTTPの内容を理解できる。L4 serviceは主にconnection情報を見る。

## 6.2 Control planeとData plane

### Control plane

設定・管理を行う側。

- CreateBucket
- RunInstances
- PutRule
- IAM policy変更
- Route table変更

### Data plane

実際の業務データが流れる側。

- S3 GetObject / PutObject
- ALBを通るHTTP request
- DynamoDB item read/write
- SQS message送受信

CloudTrailで管理APIを追う話と、VPC Flow Logsでnetwork flowを見る話を混同しない。

## 6.3 Management planeとRuntime

- Management：resourceを作る、設定する、監視する
- Runtime：作成済みresource上でapplicationやdata処理を動かす

IaCはmanagement側を自動化する。Application codeはruntime側で動く。

## 6.4 Edge、Network、Compute、Data

構成を次の四層へ分けると理解しやすい。

```text
Edge
  CloudFront / WAF / Global Accelerator
Network entry
  Route 53 / ALB / NLB / API Gateway
Compute
  EC2 / ECS / EKS / Lambda
Data
  S3 / RDS / DynamoDB / ElastiCache
```

一つのserviceが複数の役割を持つことはあるが、まず主役割で分類する。

---

# 7. パブリックとプライベート

## 7.1 Publicは「誰でも利用可能」と同じではない

Public IPやpublic endpointを持っていても、IAM、Security Group、WAF、authenticationでアクセスを制限できる。

Publicは主に、インターネットから到達可能な経路・名前・addressを持つという意味で使われる。

## 7.2 Privateは「安全が保証される」と同じではない

Private subnet内でも、過剰なIAM権限、脆弱なapplication、誤ったrouteやSecurity Groupがあれば危険である。

Privateは通信経路の性質であり、セキュリティ全体の完成を意味しない。

## 7.3 Public subnetとPrivate subnet

Subnet自体にpublic/private属性があるのではなく、一般にはrouteで区別する。

- Public subnet：Internet Gatewayへのrouteを持ち、resourceがpublic addressを使える
- Private subnet：Internet Gatewayへの直接routeを持たない

Private subnetから外向きinternetへ出る場合、NAT Gatewayなどを使う。

## 7.4 Public endpointとVPC endpoint

- Public endpoint：serviceのpublic DNS endpointへ接続
- VPC endpoint：internet gatewayやNATを経由せず、VPCからserviceへ接続

Endpointは「接続先を表す入口」であり、すべて同じ実装ではない。

- Gateway Endpoint：S3、DynamoDB向けroute table連携
- Interface Endpoint：ENIとPrivate IPを使うPrivateLink方式

---

# 8. 同期と非同期

## 8.1 同期

Callerが処理結果を待つ。

```text
Client → API → DB
Client ← responseを待つ
```

長所：結果がすぐ分かる。
短所：後段の遅延や障害がcallerへ伝わりやすい。

## 8.2 非同期

Callerは処理要求を渡し、最終処理を待たずに先へ進む。

```text
Client → API → Queue
Client ← accepted

Worker → Queueから取得 → 処理
```

長所：速度差、障害、急増を吸収しやすい。
短所：最終結果の通知、再試行、重複、順序、監視が必要。

## 8.3 EventとMessage

- Event：何かが起きたという事実
- Command/Message：何かをしてほしいという要求

実際のサービスでは境界が曖昧なこともあるが、設計意図を分けると理解しやすい。

```text
OrderCreated = 注文が作成された
ShipOrder    = 注文を発送せよ
```

## 8.4 非同期は高速化と同じではない

Userへ早く応答できても、後段処理の総時間が短くなるとは限らない。非同期化は、待ち時間の見せ方と障害境界を変える。

---

# 9. ステートレスとステートフル

## 9.1 Stateとは何か

Stateは、次の処理に必要な「前回までの状態」である。

- login session
- shopping cart
- workflow progress
- database record
- uploaded file
- sequence number

## 9.2 Stateless application

Application instance自身に継続状態を持たせない。

```text
ALB
  → App A
  → App B

SessionはElastiCache / DynamoDB / DBへ保存
```

どのinstanceへrequestが届いても処理しやすく、scale outやreplacementが容易になる。

## 9.3 Stateful service

継続状態を持つ。

- RDS
- DynamoDB
- EBS volume
- EFS
- ElastiCache cluster

Statefulだから悪いのではない。**状態をどこへ持たせるかを意図的に決める**ことが重要である。

## 9.4 Sticky session

同じclientを同じtargetへ寄せることで、instance内sessionを維持しやすくする。

ただし、target障害やscale時の制約になるため、可能ならstateを外部化する。

---

# 10. マネージドとサーバーレス

## 10.1 Managed service

AWSが一部の運用責任を引き受けるservice。

例：RDSでは、物理server、基本的なDB installation、backup機能、failure detectionなどをAWSが管理する。利用者はschema、query、parameter、capacity、securityなどを管理する。

「managed」は「利用者の設定が不要」という意味ではない。

## 10.2 Serverless

Serverが存在しないのではない。利用者がserver単位のprovisioningや管理を意識しにくい実行モデルである。

代表的な性質：

- requestや実行量に応じた課金
- 自動的なcapacity管理
- server OS管理をしない
- service固有の上限と実行モデルがある

## 10.3 Serverlessを選ぶ判断

向く：

- 変動が大きい
- event-driven
- idle時間が長い
- 運用人数が少ない
- 短時間で独立した処理

注意：

- 長時間処理
- 特殊runtime
- 常時高負荷
- connection管理
- cold startやservice quota
- vendor固有機能への依存

## 10.4 運用責任の移動

```text
EC2
  利用者: OS、patch、runtime、scaling、application

ECS on Fargate
  利用者: container image、task、application、scaling policy

Lambda
  利用者: function code、configuration、event、concurrency
```

「運用負荷が小さい」とは、運用がゼロではなく、責任範囲が上位へ移ることである。

---

# 11. 可用性・冗長化・複製・バックアップ・DR

## 11.1 Availability

必要なときにserviceが利用できる性質。

可用性を上げるには、単一障害点を減らし、障害検知と切替を自動化する。

## 11.2 Redundancy

同じ役割を複数用意すること。

- 複数EC2
- 複数AZ
- 複数network connection
- 複数DNS server

冗長なresourceがあっても、切替経路がなければ可用性は上がらない。

## 11.3 Replication

Dataを複製し、複数の場所へ持つこと。

確認項目：

- 同期か非同期か
- 一方向か双方向か
- read可能か
- write可能か
- lagがあるか
- conflictが起きるか

## 11.4 Backup

破損、誤削除、ransomware、過去時点復元に備える。

確認項目：

- retention
- immutableか
- cross-account / cross-regionか
- restore testをしているか
- RTO内に復元できるか

## 11.5 Disaster Recovery

大規模障害後に業務を再開する計画。

DRはdata copyだけではない。

```text
Infrastructure
Application
Data
Network / DNS
Identity / Secret / Certificate
Operation procedure
Test
```

## 11.6 Multi-AZとMulti-Region

- Multi-AZ：一つのRegion内でAZ障害へ備える
- Multi-Region：Region規模障害、地理的低遅延、規制等へ備える

Multi-Regionは、data replication、routing、deployment、secret、observabilityを複数Regionで設計するため、複雑性とcostが増える。

## 11.7 RTOとRPO

- RTO：何時間以内に復旧するか
- RPO：何時間分までdata lossを許容するか

```text
RTOが短い
  → 待機環境をより多く常時稼働
  → costが上がりやすい

RPOが短い
  → 頻繁または継続的なreplication
  → network、storage、整合性設計が増える
```

---

# 12. 性能を表す言葉

## 12.1 Latency

一つの処理が完了するまでの時間。

例：一回のAPI requestが50ms。

## 12.2 Throughput

一定時間に処理できる量。

例：1秒あたり1,000 request、毎秒500MB。

Latencyが低くても、同時処理数が少なければthroughputは低いことがある。

## 12.3 IOPS

一秒あたりのI/O operation数。小さなrandom I/Oが多いworkloadで重要になる。

## 12.4 Bandwidth

通信路が運べるdata量の上限。Throughputは実際に達成した量として使われることが多い。

## 12.5 Concurrency

同時に進行している処理数。

Lambda concurrency、DB connection数、thread数は関連するが同じものではない。

```text
Lambda 1,000 concurrent executions
  → 各executionがDB connectionを1本作る
  → DBへ1,000 connections
```

ここでRDS Proxyやconnection reuseが必要になる。

## 12.6 Capacity

保持・処理できる総量や能力。

- storage capacity
- compute capacity
- queue depth
- provisioned capacity

## 12.7 Burst

短時間だけ通常より大きな処理量を許容すること。

Burst性能を常時性能と誤解しない。

## 12.8 Bottleneck

全体性能を制限している最も細い部分。

```text
Client → ALB → ECS → RDS
                    ↑
                DB connectionが上限
```

ECSを増やしても、DBがbottleneckなら全体性能は上がらない。

---

# 13. 認証・認可・暗号化

## 13.1 Identity

誰・何であるかを表す。

- human user
- IAM role
- application
- device
- AWS service

## 13.2 Credential

Identityを証明するための情報。

- password
- access key
- temporary credential
- certificate
- token

Identityとcredentialを同じ意味で使わない。

## 13.3 Authentication

Credentialを検証し、identityを確認する。

```text
User + password/MFA
  → Cognito / IdP
  → authentication success
  → token
```

## 13.4 Authorization

Authenticated identityへ許可されたoperationを決める。

```text
Principal
  + Action
  + Resource
  + Condition
  + Policy evaluation
  → Allow / Deny
```

## 13.5 Role assumption

STSを通じて一時的にroleの権限を取得する。

```text
Principal
  → sts:AssumeRole
  → temporary credentials
  → target account/resourceへaccess
```

Roleを「user group」と考えない。Roleは引き受けるidentityとpermissionの組である。

## 13.6 Encryption key

Dataを暗号化・復号するためのkey。

KMSでは、key material、KMS key、key policy、grant、data keyなど複数概念がある。

Envelope encryptionの基本：

```text
KMS key
  → data keyを生成/暗号化
Data key
  → 大きなdata本体を暗号化
Encrypted data key
  → dataと一緒に保存
```

KMS keyで巨大dataを直接すべて処理するという理解ではない。

## 13.7 TLS

通信相手の証明と通信暗号化を行うprotocol。

TLSがあるからapplication userのauthorizationまで完了するわけではない。

## 13.8 Least privilege

必要なidentityへ、必要なactionを、必要なresourceに、必要なconditionでだけ許可する。

「ReadOnlyAccessを付けたから安全」ではなく、業務に必要なresource範囲まで絞る。

> 深掘り: [Access Control and Encryption Deep Dive](comparisons/access-control-and-encryption.md)、[IAM境界](comparisons/iam-boundaries-scp-condition-deep-dive.md)

---

# 14. 再試行・冪等性・タイムアウト

## 14.1 Timeout

どこまで待ったら失敗と判断するか。

Timeoutが長すぎるとresourceを占有し、短すぎると正常処理を失敗扱いする。

## 14.2 Retry

一時的失敗へ再度処理を試す。

Retryすべき例：

- transient network error
- throttling
- 一時的なservice unavailable

Retryしてはいけない、または修正が必要な例：

- validation error
- permission denied
- 存在しないresource
- 恒久的なconfiguration error

## 14.3 Exponential backoff

再試行間隔を徐々に長くし、障害中のserviceへ負荷を集中させない。

## 14.4 Jitter

再試行時刻へばらつきを加え、多数clientが同時再試行するthundering herdを避ける。

## 14.5 Idempotency

同じrequestを複数回実行しても、意図した最終結果が一回実行時と同じになる性質。

```text
Payment request ID = 123

1回目 → 課金
retry → ID 123は処理済みなので二重課金しない
```

SQS Standard、Lambda async、network retryなど、重複が起こり得るsystemでは重要である。

## 14.6 DLQ

規定回数失敗したmessageを隔離するQueue。

DLQへ送るだけでは解決ではない。

- alert
- root cause調査
- data修正
- replay手順
- retention

まで設計する。

## 14.7 Circuit breaker

失敗中のdependencyへrequestを送り続けず、一時的に遮断する。

Retryと反対ではない。Retry回数を制御し、一定条件で回路を開く。

---

# 15. AWSの代表的な一文を展開する

## 15.1 「CloudFrontはedgeでcacheする」

### 圧縮された説明

> CloudFrontはコンテンツをエッジロケーションへキャッシュし、低レイテンシで配信する。

### 展開

```text
ViewerがCloudFront distributionへHTTP requestを送る。
CloudFrontはcache keyを作り、edge cacheを確認する。
Cache hitなら、originへ接続せずresponseを返す。
Cache missなら、S3やALBなどのoriginへrequestを送る。
Origin responseをTTLやcache policyに従って保存する。
次のrequestではedgeから返せるため、origin loadとnetwork distanceを減らせる。
```

### 何ではないか

- 任意のTCP/UDPをcacheするserviceではない
- application codeを実行する場所ではない
- origin dataの正本ではない

### 代償

- cache invalidation
- stale data
- cache key設計
- dynamic contentの扱い

## 15.2 「ALBはtrafficを分散する」

### 展開

```text
ClientがALB DNS nameへ接続する。
Listenerがport/protocolを受ける。
Ruleがhost/path/headerを評価する。
Target groupを選び、healthy targetへrequestをforwardする。
Targetのhealth checkが失敗すれば、新規requestの送り先から外す。
```

### 何ではないか

- DB queryを分散するserviceではない
- application stateを自動共有するserviceではない
- DNS routing policyの代替ではない

## 15.3 「SQSは疎結合にする」

### 展開

```text
Producerはconsumerを直接呼ばず、messageをQueueへ保存する。
Queueはconsumerが取得するまでmessageを保持する。
Consumerは自身の処理能力に合わせてmessageを取得する。
Consumerが停止してもproducerはQueueへ送信できる。
Producerの急増はQueue depthとして蓄積される。
```

### 代償

- eventual completion
- duplicateへの対応
- orderの設計
- DLQと監視

## 15.4 「EventBridgeからECS taskを起動できる」

### 展開

```text
Schedule時刻またはevent patternが成立する。
EventBridge Scheduler / Ruleがtarget actionを実行する。
TargetはECS RunTask APIである。
ECSがtask definitionに基づきtaskを配置する。
FargateまたはEC2 capacity上でcontainerが起動する。
```

### なぜAPI Gatewayが不要か

API Gatewayは、外部clientからHTTP APIを受ける入口である。この構成では、EventBridgeがAWS APIを直接呼び出すため、外部HTTP endpointを公開する必要がない。

```text
不要な経路:
EventBridge → API Gateway → Lambda → ECS RunTask

直接経路:
EventBridge → ECS RunTask
```

### API Gatewayが必要になる場合

- 人や外部applicationからHTTPで任意実行する
- authentication、rate limit、request validationが必要
- public/private APIとして管理する

## 15.5 「PrivateLinkはprivate connectivityを提供する」

### 展開

```text
Service providerがNLB配下のserviceをEndpoint Serviceとして公開する。
Consumer VPCがInterface Endpointを作る。
Consumer subnet内にPrivate IPを持つENIが作られる。
ConsumerはそのPrivate IP / private DNS nameへ接続する。
Trafficはpublic internetへ出ず、provider serviceへ到達する。
```

### 何ではないか

- VPC全体を相互routingする仕組みではない
- ConsumerからProvider内の任意IPへ接続する仕組みではない
- Transit Gatewayのようなnetwork hubではない

## 15.6 「RDS Multi-AZは高可用性を提供する」

### 展開

```text
Primary DB instanceがapplication requestを処理する。
Standbyが別AZに維持される。
AWSがinstance、storage、AZなどのfailureを検知する。
必要に応じてStandbyへfailoverする。
DB endpointを通じてapplicationが新Primaryへ再接続する。
```

### 何ではないか

- Read scalingのためのRead Replicaと同じではない
- 過去時点復元のBackupと同じではない
- Region障害へ自動的に備えるMulti-Region構成ではない

## 15.7 「Aurora Global Databaseはcross-region DRを支援する」

### 展開

```text
Primary Regionでwriteする。
Aurora storage layerの変更がSecondary Regionへ複製される。
Secondary Regionでreadを提供できる。
Primary Region障害時は、運用手順またはservice機能でSecondaryを昇格する。
Application trafficとDNSを新Primaryへ切り替える。
```

### 確認すること

- planned switchoverかunplanned failoverか
- RPO/RTO
- write endpoint切替
- secret、application、networkもSecondaryにあるか

DBだけ複製してもDRは完成しない。

## 15.8 「SCPはpermission guardrailである」

### 展開

```text
AdministratorがSCPをRoot / OU / Accountへattachする。
IAM principalがAWS APIを実行する。
AWSはidentity policy、resource policy、SCPなどを評価する。
SCPの許可範囲外、または明示Denyなら実行できない。
IAM policyでAllowされていてもSCPを越えられない。
```

### 何ではないか

- permissionを直接付与するpolicyではない
- management accountのすべての操作を同じように制限するものではない
- resource policyの代替ではない

## 15.9 「RDS Proxyはconnectionをpoolする」

### 展開

```text
LambdaやApplicationがRDS Proxyへconnectionを作る。
Proxyは多数のclient connectionを受ける。
Proxyはより少数のDB connectionを再利用する。
Databaseへのconnection stormを抑える。
Failover時のconnection handlingを改善する。
```

### 何ではないか

- Query result cacheではない
- Read Replicaではない
- SQL performanceを無条件に高速化するものではない

## 15.10 「DynamoDB Global TablesはActive-Activeである」

### 展開

```text
複数Regionにreplica tableを持つ。
各Regionでlocal read/writeを受け付ける。
変更が他Regionへreplicateされる。
同じitemへ競合writeが発生し得るため、conflict resolutionとapplication behaviorを理解する。
```

### 判断

Global userの低latency writeやRegion resilienceに向く。一方、強いglobal transactionや複雑なrelational queryを必要とする場合は別設計を検討する。

---

# 16. 良い説明を作るテンプレート

個別サービスの説明を書くときは、次の順序を使う。

## 16.1 最小テンプレート

```text
一言でいうと:
誰が使う:
何を受け取る:
何をする:
何を出す / どんな状態になる:
なぜ必要:
何と比較する:
何ではない:
制約 / 代償:
小さな構成図:
```

## 16.2 SAP向けテンプレート

```text
課題:
強い制約:
現状:
候補:
採用案:
通信経路:
データの正本:
障害時の動作:
運用責任:
コスト要因:
不採用案と理由:
```

## 16.3 用語向けテンプレート

```text
語源 / 直訳:
ITでの意味:
AWSでの代表例:
誰が主語か:
何を対象にするか:
似た語との違い:
問題文での出方:
誤解しやすい点:
```

## 16.4 比較向けテンプレート

```text
共通点:
違う軸:
Aが見る情報:
Bが見る情報:
Aが保持する状態:
Bが保持する状態:
Aを選ぶ制約語:
Bを選ぶ制約語:
同時に使う構成:
```

---

# 17. 分かりにくい説明の典型

## 17.1 サービス名を別のサービス名で説明する

悪い例：

> EventBridgeはSNSやSQSのようなevent-driven serviceである。

SNSやSQSを知らなければ説明にならない。

改善：

> EventBridgeは、発生したeventの内容をruleと照合し、一致したeventを指定したtargetへ送るrouting serviceである。Queueのようにconsumerの処理待ちを主目的としてmessageを蓄積するserviceではない。

## 17.2 メリットだけを書く

悪い例：

> Lambdaは運用負荷が低く、scalableでcost efficientである。

改善：

- 何を管理しなくてよいか
- 何に応じてscaleするか
- どの課金特性で安くなるか
- どのworkloadでは不利か

まで書く。

## 17.3 「自動」を説明しない

> 自動でfailoverする。

確認すべきこと：

- 何を監視するか
- 誰が障害を判定するか
- 何へ切り替えるか
- 接続中sessionはどうなるか
- applicationは再接続が必要か
- どの程度の時間がかかるか

## 17.4 「secure」を一語で済ませる

Securityは次へ分ける。

- identity
- authentication
- authorization
- network path
- encryption
- logging
- detection
- response
- data retention

## 17.5 「high availability」と「disaster recovery」を混ぜる

Multi-AZでAZ障害へ備える説明を、Region障害のDRと同一視しない。

## 17.6 構成図に矢印だけあり、矢印の意味がない

悪い図：

```text
S3 → EventBridge → Lambda → RDS
```

良い図：

```text
S3 Object Created event
  → EventBridge ruleでevent pattern照合
  → Lambda async invocation
  → LambdaがRDS Proxy経由でSQL write
```

矢印には、HTTP、event、message、replication、DNS、role assumptionなどの意味を付ける。

## 17.7 正本が分からない

Cache、replica、backup、indexを説明するときは、正本dataがどこかを必ず書く。

```text
RDS = source of truth
ElastiCache = derived cache
S3 backup = restore copy
OpenSearch = search index
```

---

# 18. 説明を読んだときの確認問題

次の一文を読んだら、答えられるか確認する。

## 18.1 「NAT GatewayによりPrivate subnetからinternetへ接続できる」

- 誰がconnectionを開始するか
- inbound internet connectionも開始できるのか
- route tableはどこを向くか
- NAT Gatewayはどのsubnetに置くか
- Internet Gatewayは必要か
- AZ障害とcostをどう考えるか

## 18.2 「API GatewayはAPIを管理する」

- APIを管理するとは具体的に何か
- ALBと違い、何を提供するか
- authentication、authorization、throttlingはどこで行うか
- backendはLambdaだけか
- private APIは作れるか

## 18.3 「ElastiCacheでdatabase負荷を減らす」

- 何をcacheするか
- cache hit/miss時のflowは何か
- TTLはどうするか
- update時にcacheをどう無効化するか
- cache停止時も正しく動くか

## 18.4 「Auto Scalingで可用性を高める」

- 何を増減するか
- どのAZへ配置するか
- health checkは何を見るか
- scalingとreplacementは何が違うか
- load balancerは必要か

## 18.5 「Cross-account roleで安全にaccessする」

- 誰がroleをassumeするか
- Trust policyは誰を信頼するか
- Permission policyは何を許可するか
- External IDやconditionは必要か
- Temporary credentialの期限は何か

## 18.6 「CloudTrailで監査する」

- どのAPI eventを記録するか
- Management eventとData eventの違いは何か
- Logをどのaccountへ集約するか
- 改ざん・削除をどう防ぐか
- 検知と対応をどう自動化するか

## 18.7 「S3は高耐久である」

- 耐久性と可用性は何が違うか
- 誤削除は耐久性だけで防げるか
- Versioning、Object Lock、replicationは何を解決するか
- Applicationのread/write availabilityは別に考えたか

## 18.8 「Direct Connectは安定したnetwork接続を提供する」

- internet VPNと何が違うか
- 暗号化は自動で保証されるか
- 冗長化はどのlocation/deviceで行うか
- VIFとDX Gatewayは何をつなぐか
- backup pathは必要か

## 18.9 「Step Functionsでworkflowを作る」

- Event routingだけでなく、どの状態を持つか
- retry、catch、timeoutをどこで定義するか
- StandardとExpressの実行特性は何か
- compensationが必要か

## 18.10 「CloudFormationでinfrastructureを自動化する」

- templateが定義するdesired stateは何か
- Change Setは何を見るか
- Driftとは何か
- StackSetsは何を複数account/Regionへ展開するか
- Application deploymentまで同じ責任か

---

# 19. このリポジトリでの使い方

## 推奨読書順

1. **このページ**で説明文の分解方法を覚える
2. [AWS SAP設計読本](SAP_DESIGN_READER.md) で設計判断の流れを学ぶ
3. [LEARNING_PATH](LEARNING_PATH.md) に沿って分野別に深める
4. [SERVICES_INDEX](SERVICES_INDEX.md) から個別serviceを調べる
5. `comparisons/` で似た選択肢を比較する
6. `practice/` で長文問題を解く

## 分からないときの戻り先

| 詰まった内容 | 戻るページ |
|---|---|
| Web server、Tomcat、Nginx、Proxy | [Web Runtime and Proxy Terms](glossary/web-runtime.md) |
| User Pool、Identity Pool、Connection Pool | [Pool Terms](glossary/pool-terms.md) |
| Route、SG、NACL、NAT、Endpoint | [Networking Foundations](comparisons/networking-foundations-deep-dive.md) |
| RDS endpoint、Reader、Proxy、Connection | [RDS/Aurora Connection](comparisons/rds-aurora-connection-deep-dive.md) |
| AuthN、AuthZ、KMS、TLS | [Access Control and Encryption](comparisons/access-control-and-encryption.md) |
| SCP、Permission Boundary、Session Policy | [IAM Boundaries](comparisons/iam-boundaries-scp-condition-deep-dive.md) |

## 最終的に目指す説明

悪い説明：

> EventBridgeはevent-driven architectureに使うserviceです。

良い説明：

> EventBridgeは、AWS serviceやapplicationから受け取ったeventを、event patternとruleで評価し、一致したものをLambda、ECS、Step Functionsなどのtargetへ送る。送信側は受信側のendpointを直接知らなくてよいため依存を減らせる。一方、consumerの処理速度差を吸収する永続Queueが必要ならSQSを組み合わせる。定刻にECS taskを起動するだけなら、EventBridgeからECS RunTaskを直接呼べるため、外部HTTP API要件がない限りAPI Gatewayは不要である。

このレベルまで展開できれば、単語を知っているだけでなく、構成・通信・責任・トレードオフを理解している。

---

# まとめ

AWSの説明を理解するために、最初からすべてのserviceを暗記する必要はない。説明を次へ分解する。

```text
誰が
  → 何を
  → どこへ
  → いつ
  → どうやって
  → なぜ
  → どんな代償で
```

そして、必ず次を確認する。

- 通信経路は何か
- dataの正本はどこか
- 状態は誰が持つか
- 障害時に何が起きるか
- 誰がどこまで運用するか
- 何と比較して選ぶのか
- 何ではないのか

**説明とは、用語を増やすことではない。隠れている処理順序と因果関係を見える形へ戻すことである。**
