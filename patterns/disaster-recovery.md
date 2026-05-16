# Pattern: Disaster Recovery

## 4つの DR 戦略と定量比較

SAP-C02 では「RTO/RPO の要件」と「コスト」から最適戦略を選ばせる問題が頻出。DR はサービス名暗記ではなく、**許容停止時間、許容データ損失、常時稼働コスト、切替自動化レベル**のトレードオフとして読む。

| 戦略 | 目安RTO | 目安RPO | コスト | 概要 |
|---|---:|---:|---|---|
| **Backup & Restore** | 数時間〜24h | 数時間〜24h | 最低 | バックアップのみ保持し、障害時に再構築・復元 |
| **Pilot Light** | 数十分〜1h | 数分〜15分 | 低〜中 | データ層など最小限のコアだけDR側で常時稼働 |
| **Warm Standby** | 数分〜15分 | 数秒〜数分 | 中〜高 | スケールダウン版の本番環境をDR側で常時稼働 |
| **Multi-Site Active/Active** | ほぼゼロ〜数分 | ほぼゼロ〜数秒 | 最高 | 複数リージョンで同時に本番トラフィックを処理 |

> 数値は設計上の目安。実際の RTO/RPO はアプリ構成、データストア、フェイルオーバー手順、DNS TTL、運用テストの成熟度に依存する。

### コスト vs RTO/RPO トレードオフ図

```
コスト
 高 |                              ★ Multi-Site
    |                    ★ Warm Standby
    |          ★ Pilot Light
 低 | ★ Backup & Restore
    └─────────────────────────────→ 復旧速度
      遅い RTO/RPO            速い RTO/RPO
```

---

## 各戦略の詳細

### 1. Backup & Restore

**適合条件**: RTO/RPO が数時間以上許容される、コスト最優先、重要度が相対的に低いシステム。

- S3 / AWS Backup / EBS Snapshot / RDS Snapshot にデータを退避。
- バックアップをクロスリージョンコピーする。
- 障害時に CloudFormation / CDK / Terraform で環境を再構築し、スナップショットを復元。
- DRリージョンの常時稼働リソースがほぼ不要なため最安。

```
本番リージョン ──Snapshot / Backup──→ S3 / AWS Backup Vault (Cross-Region Copy)
                                                     │
                                                     ▼
                                              障害時にDRリージョンへ復元
```

**SAPの読み方**: `minimum cost`, `downtime of several hours acceptable`, `backup retention` が出たら候補。

---

### 2. Pilot Light

**適合条件**: RTO 1時間以内、RPO 数分〜15分程度、コストを抑えつつデータ層は常に最新に近づけたい。

- データ層や認証基盤など、復旧に時間がかかるコア部分だけDR側で常時稼働。
- アプリケーション層は AMI / コンテナイメージ / Auto Scaling 設定のみ用意し、普段は停止または最小化。
- 障害時に Route 53 Failover、Auto Scaling、ECS Service Desired Count 変更などで起動・拡張する。

```
本番リージョン                    DR リージョン
  ALB → ECS/EC2 ─────────────→  AMI / ECR / IaC（待機）
  RDS Primary ──Replication──→  RDS Read Replica（常時稼働）
```

**SAPの読み方**: `cost-effective`, `core services running`, `RTO less than one hour` が出たら候補。

---

### 3. Warm Standby

**適合条件**: RTO 15分以内、RPO 数秒〜数分、重要業務システム。コストより復旧速度が重要。

- DRリージョンにスケールダウン版のフルスタックを常時稼働。
- ALB、ECS/EC2、RDS/Aurora Replica、ElastiCache、必要なVPC構成を事前に用意。
- 障害時は Route 53 / Global Accelerator で切替え、Auto Scaling で本番規模に拡張。

```
本番リージョン（フルサイズ）          DR リージョン（スケールダウン）
  ALB → ECS x10 → DB Primary ──→  ALB → ECS x2 → DB Replica
                                    │
                                    └─ 障害時に x10 へスケールアップ
```

**SAPの読み方**: `critical application`, `low RTO`, `scaled-down environment running` が出たら候補。

---

### 4. Multi-Site Active/Active

**適合条件**: RTO/RPO を極小化したい、グローバルユーザーに低レイテンシを提供したい、常時複数リージョン運用のコストと複雑性を許容できる。

- 複数リージョンでアプリケーションを同時に稼働。
- Route 53 レイテンシーベース / 加重ルーティング、または AWS Global Accelerator でトラフィックを分散。
- データ層はサービス特性に応じて選択する。
  - **DynamoDB Global Tables**: マルチリージョン・マルチアクティブ書き込みに向く。
  - **Aurora Global Database**: 基本は「1つの Primary Region に書き込み、Secondary Region は読み取り専用」。障害時に switchover / failover で Primary を移す。write forwarding は Secondary から Primary に書き込みを転送する機能であり、一般的な双方向multi-writeとは別物。

```
Route 53 / Global Accelerator
    ├──→ us-east-1:        ALB → ECS → Aurora Primary（Write）
    └──→ ap-northeast-1:   ALB → ECS → Aurora Secondary（Read-only）
                                ↑ Primary障害時に昇格 / 切替

DynamoDB Global Tables の場合:
    us-east-1  Table  ⇄  ap-northeast-1 Table
    複数リージョンから書き込み可能
```

**SAPの読み方**: `active-active writes` なら Aurora Global Database だけでなく DynamoDB Global Tables などを検討する。`relational database`, `global reads`, `low RPO`, `regional failover` なら Aurora Global Database が候補。

---

## 試験の選択基準（意思決定ツリー）

```
RTO/RPO の要件は?
    │
    ├── 数時間以上でよい
    │      └─ Backup & Restore（コスト最小）
    │
    ├── RTO < 1h, RPO < 15分
    │      └─ Pilot Light（コスト優先 + データ層は常時同期）
    │
    ├── RTO < 15分, RPO < 数分
    │      └─ Warm Standby（スケールダウン版を常時稼働）
    │
    └── RTO/RPO ≈ 0、またはグローバル低レイテンシ
           └─ Multi-Site Active/Active（最高コスト・高複雑性）
```

### データストア別の選択

| 要件 | 候補 | 注意点 |
|---|---|---|
| オブジェクトを別リージョンへ複製 | S3 CRR / SRR | 削除マーカー・既存オブジェクト複製の扱いに注意 |
| RDSを別リージョンに待機 | Cross-Region Read Replica / Snapshot Copy | 昇格手順と接続先切替が必要 |
| Auroraで低RPOのリージョンDR | Aurora Global Database | Secondaryは基本read-only。writeはPrimaryへ |
| NoSQLでmulti-region writes | DynamoDB Global Tables | 競合解決・アプリ側設計に注意 |
| ファイルサーバ移行/同期 | DataSync / FSx replication | 継続同期か一括移行かを読む |
| 大容量オフライン移行 | Snow Family | ネットワーク転送より物理搬送が現実的な場合 |

---

## 主要サービスの DR における役割

| サービス | 役割 | 対応戦略 |
|---|---|---|
| **Route 53 Failover Routing** | ヘルスチェック + DNS 切替 | 全戦略 |
| **AWS Global Accelerator** | Anycast IP による高速なリージョン切替 | Warm, Active/Active |
| **S3 Cross-Region Replication** | オブジェクトの自動複製 | 全戦略 |
| **AWS Backup** | バックアップ一元管理・クロスリージョンコピー | Backup & Restore |
| **RDS Cross-Region Read Replica** | DB待機系・昇格 | Pilot Light, Warm |
| **Aurora Global Database** | 低レイテンシ read replica + リージョンDR | Warm, Active/Passive寄り |
| **DynamoDB Global Tables** | マルチリージョン・マルチアクティブNoSQL | Active/Active |
| **CloudFormation StackSets** | DR環境の IaC 展開 | 全戦略 |
| **Elastic Disaster Recovery / MGN** | サーバーレベルの継続レプリケーション | Pilot Light, Warm |
| **FIS** | DR手順・障害注入テスト | 全戦略 |

---

## Well-Architected: Reliability Pillar

- **RTO/RPOをビジネス要件として定義する**: 技術要件ではなく、停止損失と復旧コストから決める。
- **復旧手順を自動化する**: IaC、Runbook、SSM Automation、Route 53 / Global Accelerator を活用。
- **データ整合性を監視する**: replication lag、backup success、restore test を継続確認。
- **定期的にテストする**: FIS やゲームデーで、DR手順が実際に動くか検証。
- **DNS TTLとクライアント接続を考慮する**: 切替設計はDNSだけでは完結しない。

---

## SAP-C02 頻出問題パターン

| キーワード | 正解戦略 |
|---|---|
| `minimum cost`, `hours of downtime acceptable` | Backup & Restore |
| `RTO < 1 hour`, `cost-effective`, `core services` | Pilot Light |
| `RTO < 15 minutes`, `critical application`, `scaled-down environment` | Warm Standby |
| `zero downtime`, `global users`, `traffic served from multiple regions` | Multi-Site Active/Active |
| `relational database`, `global reads`, `low RPO`, `regional failover` | Aurora Global Database |
| `multi-region writes`, `NoSQL`, `active-active` | DynamoDB Global Tables |
| `petabytes`, `limited bandwidth`, `physical transfer` | Snow Family |
| `continuous server replication`, `rehost` | AWS Application Migration Service / Elastic Disaster Recovery |
