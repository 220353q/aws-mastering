# AWS バックアップ・ネットワーク・設計パターン整理

## 1. ALB / NLB の『前段』とは？

AWSで『前段にALBを置く』『NLBを前段に置く』という表現は、
『ユーザーからの通信を最初に受け取る入口』という意味。

例:

ユーザー → ALB → EC2

この場合、ALBが前段。

---

## 2. RESTful API と通信経路

RESTful API は HTTP/HTTPS を使った API 通信方式。

例:

スマホアプリ
 ↓ HTTPS
ALB
 ↓
ECS / Lambda / EC2
 ↓
RDS

### ALB

- HTTP/HTTPS レベルで動作
- URL パスで振り分け可能
- Cookie や Header を見られる

例:

/api/user → ECS-A
/api/admin → ECS-B

### NLB

- TCP/UDP レベルで動作
- 高速
- 通信内容を見ない
- 超低遅延

例:

ゲーム
VoIP
金融取引

### Global Accelerator

- AWS グローバルネットワークを使う
- 世界中から最短経路で接続
- リージョン障害時の切り替え

---

## 3. AWS バックアップ関連サービス

### 1. AWS Backup

複数サービスのバックアップを統合管理。

対象:

- EBS
- RDS
- DynamoDB
- EFS

ユースケース:

企業全体で統一バックアップポリシーを適用。

---

### 2. EBS Snapshot

EC2 ディスクのポイントインタイム保存。

ユースケース:

- 障害復旧
- AMI作成
- DR対策

### Fast Snapshot Restore

通常 Snapshot は lazy load。
Fast Snapshot Restore は即時高速復元。

RTO短縮に有効。

---

### 3. RDS 自動バックアップ

自動でログ・スナップショット取得。

特徴:

- Point In Time Recovery
- 自動保持

ユースケース:

DB障害復旧。

---

### 4. RDS Read Replica

読み取り専用複製。

目的:

- Read負荷分散
- レイテンシ削減

特徴:

- 非同期レプリケーションが多い
- バックアップ用途ではない

---

### 5. Multi-AZ

同期レプリケーション。

目的:

- 高可用性
- 自動フェイルオーバー

特徴:

- RPOほぼゼロ
- RTO短い

---

### 6. DynamoDB Global Tables

複数リージョンで読み書き可能。

特徴:

- Active-Active
- 低レイテンシ
- グローバル分散

---

### 7. S3 Versioning

オブジェクト履歴管理。

特徴:

- 誤削除対策
- ランサムウェア対策

---

### 8. S3 CRR

Cross Region Replication。

特徴:

- 別リージョン複製
- DR対策

注意:

バックアップ寄り。
DBのような即時読み書き用途ではない。

---

## 4. レプリケーション vs リードレプリカ

### レプリケーション

目的:

- DR
- 冗長化
- 可用性

例:

- Multi-AZ
- S3 CRR
- DynamoDB Global Tables

### リードレプリカ

目的:

- Read性能向上
- 負荷分散

例:

- RDS Read Replica

---

## 5. 同期 / 非同期

### 同期

書き込み完了まで待つ。

特徴:

- データ損失少
- 遅延増える

例:

- Multi-AZ

### 非同期

後から複製。

特徴:

- 高速
- データロス可能性

例:

- Read Replica
- S3 CRR

---

## 6. RTO / RPO

### RTO

復旧まで許容時間。

短くしたい場合:

- Multi-AZ
- Fast Snapshot Restore
- Global Accelerator

---

### RPO

許容データ損失量。

短くしたい場合:

- 同期レプリケーション
- Global Tables
- Multi-AZ

---

## 7. AWS 設計パターン

### Microservices

小さなサービスへ分割。

例:

- ECS
- Lambda
- API Gateway

---

### Event Driven

イベント発火型。

例:

S3 Upload
 ↓
EventBridge
 ↓
Lambda

---

### Serverless

サーバ管理不要。

例:

- Lambda
- DynamoDB
- API Gateway

---

### Fan-out Pattern

1つのイベントを複数処理へ配信。

例:

SNS
 ↓
SQS-A
SQS-B
Lambda

---

### CQRS

Read / Write 分離。

例:

Write → RDS Primary
Read → Read Replica
