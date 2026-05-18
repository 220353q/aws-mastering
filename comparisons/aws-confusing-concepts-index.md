# AWS 混同しやすい概念インデックス — SAP-C02横断整理

## 何のためのページか

SAP-C02では、サービス名を知っているだけでは解けない。似た名前、似た役割、似た構成要素を正しく分けて読む必要がある。

このページは、混同しやすい概念を分野別に整理した入口ページ。

---

## まず結論

SAP-C02で混同しやすい論点は、主に次の5系統に分かれる。

```text
1. Identity / Policy
   → 誰が、何に、どの上限でアクセスできるか

2. Network / Security Boundary
   → 経路、許可、検査、公開範囲を分ける

3. Endpoint / Protocol
   → 接続先、通信方式、入口サービスを分ける

4. Compute / Container / Serverless
   → 実行単位、運用責任、スケール方式を分ける

5. Data / Messaging / Analytics
   → 保存、処理、配信、分析、移行を分ける
```

---

## 1. Identity / Policy 系

読むべきページ。

- [Identity / Policy Evaluation 混同整理](identity-policy-evaluation-confusions.md)

混同しやすいもの。

| 混同 | 見分け方 |
|---|---|
| IAM Policy vs Resource Policy | 主体側の許可か、リソース側の許可か |
| SCP vs Permission Boundary | 組織ガードレールか、principal単位の上限か |
| Trust Policy vs Permission Policy | 誰がRoleを引き受けられるか、Roleが何をできるか |
| KMS Key Policy vs IAM Policy | KMS key自体の利用許可が必要か |
| Endpoint Policy vs IAM Policy | endpoint経由の操作制限か、principal権限か |
| Session Policy vs Permission Boundary | 一時認証情報のセッション制限か、principalの最大権限か |

---

## 2. Network / Security Boundary 系

読むべきページ。

- [Network / Routing / Security Boundary 混同整理](network-routing-security-confusions.md)
- [AWS Gateway Services and Terms](aws-gateways.md)
- [Networking Foundations Deep Dive](networking-foundations-deep-dive.md)
- [Security Group / NACL / Firewall Decision Guide](network-security-boundaries.md)

混同しやすいもの。

| 混同 | 見分け方 |
|---|---|
| Route Table vs Security Group | 経路か、許可か |
| Security Group vs NACL | ENI単位のstateful Allowか、Subnet単位のstateless Allow/Denyか |
| WAF vs Network Firewall | L7 Web保護か、VPC内ネットワーク検査か |
| NAT Gateway vs Internet Gateway | Private subnetの外向きか、VPCとInternetの出入口か |
| Transit Gateway vs PrivateLink | 広いネットワーク接続か、サービス単位接続か |
| Global Accelerator vs CloudFront | TCP/UDP入口最適化か、HTTP CDN/cacheか |

---

## 3. Endpoint / Protocol 系

読むべきページ。

- [AWS Endpoint / ENI / VIF 完全整理](endpoints-eni-vif.md)
- [Endpoint / ENI / VIF / PrivateLink 理解補完レジュメ](endpoints-eni-vif-remedial-resume.md)
- [HTTP / HTTPS / WebSocket / MQTT / gRPC / TCP / UDP とAWS入口サービス](protocols-and-aws-entrypoints.md)

混同しやすいもの。

| 混同 | 見分け方 |
|---|---|
| Aurora Endpoint vs VPC Endpoint | DB接続先DNSか、AWSサービスへのprivate接続口か |
| Gateway Endpoint vs Interface Endpoint | Route Table型か、ENI型か |
| ENI vs VIF | VPC内の仮想NICか、Direct Connect上の論理IFか |
| CloudFront vs API Gateway | CDN/edge/cacheか、API管理か |
| API Gateway vs IoT Core | 汎用APIか、IoTデバイス通信か |
| WebSocket vs MQTT | アプリ双方向通信か、IoT Pub/Subか |

---

## 4. Compute / Container / Serverless 系

読むべきページ。

- [Compute / Container / Serverless 混同整理](compute-container-serverless-confusions.md)

混同しやすいもの。

| 混同 | 見分け方 |
|---|---|
| EC2 vs ECS vs EKS vs Lambda | 実行単位と運用責任で分ける |
| ECS on EC2 vs Fargate | EC2管理するか、タスク実行基盤をAWSに任せるか |
| ECS vs EKS | AWSネイティブコンテナか、Kubernetes標準か |
| Lambda vs Fargate | 関数実行か、コンテナタスク実行か |
| Task Role vs Execution Role | アプリが使う権限か、ECSが起動に使う権限か |
| Auto Scaling vs Load Balancing | 台数を増減するか、通信を振り分けるか |

---

## 5. Data / Messaging / Analytics 系

読むべきページ。

- [Data / Messaging / Analytics 混同整理](data-messaging-analytics-confusions.md)
- [EventBridge vs SNS vs SQS vs Step Functions vs Amazon MQ](messaging-eventing.md)
- [Analytics / Data Lake Service Selection](analytics-data-lake.md)

混同しやすいもの。

| 混同 | 見分け方 |
|---|---|
| SQS vs SNS vs EventBridge | Queueか、通知fanoutか、イベントルーティングか |
| Kinesis vs SQS | ストリーム処理か、キュー処理か |
| Step Functions vs EventBridge | ワークフロー制御か、イベントルーティングか |
| Aurora vs DynamoDB | RDBトランザクションか、キー値/ドキュメントNoSQLか |
| Redshift vs Athena | DWHか、S3上のサーバーレスSQLか |
| Glue vs Lake Formation vs DataZone | ETL/catalogか、権限制御か、データ発見/共有か |
| DMS vs DataSync vs MGN | DB移行か、ファイル転送か、サーバー移行か |

---

## 読み方の順番

初学者は次の順で読む。

```text
1. endpoints-eni-vif-remedial-resume.md
2. endpoints-eni-vif.md
3. protocols-and-aws-entrypoints.md
4. identity-policy-evaluation-confusions.md
5. network-routing-security-confusions.md
6. compute-container-serverless-confusions.md
7. data-messaging-analytics-confusions.md
```

SAP-C02の長文問題で迷ったら、まず次の問いに戻る。

```text
これは、権限の問題か？
これは、経路の問題か？
これは、入口サービスの問題か？
これは、実行基盤の問題か？
これは、データの保存/処理/分析の問題か？
```

---

## 最短暗記

```text
IAMは「誰が何をできるか」
Networkは「どこを通るか、どこで止めるか」
Endpointは「どこへ接続するか」
Computeは「どこで実行するか」
Dataは「どこに置き、どう流し、どう読むか」
```
