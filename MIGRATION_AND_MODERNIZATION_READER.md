# AWS Migration and Modernization Reader

> MigrationはServerをAWSへコピーする作業ではない。**Portfolioを評価し、依存関係を把握し、移行戦略を選び、Waveを組み、CutoverとRollbackを実行し、移行後にModernizeするProgram**である。

SAP-C02 Domain 4では、MGN、DMS、DataSyncなどの名前だけでなく、どのWorkloadを、どの順番で、どの停止時間で、どのTarget architectureへ移すかを判断する。

```text
Discover
  → Assess
  → Mobilize
  → Migrate
  → Validate
  → Cut over
  → Stabilize
  → Modernize
```

---

# 1. Migrationの三つの層

## Portfolio layer

- 何があるか
- 何を移すか
- 何を廃止するか
- どの順番か
- Business valueは何か

## Workload layer

- Application dependency
- Data
- Identity
- Network
- RTO / RPO
- Migration strategy

## Execution layer

- Replication
- Test
- Cutover
- Rollback
- Validation
- Hypercare

個別Serviceは主にExecution layerを支援する。Portfolio判断は別途必要である。

---

# 2. Discovery

## 集める情報

- Server inventory
- OS / version
- CPU / memory / storage
- Utilization
- Installed software
- Network connection
- Database
- File share
- Batch schedule
- Business owner
- Technical owner
- SLA
- Compliance
- License

## Dependency

```text
Web
  → App
  → Database
  → File share
  → LDAP
  → Payment API
  → Monitoring
```

Server単位で移すと、取り残したDependencyによりApplicationが動かない。

## Tools

- Migration Hub
- Application Discovery Service
- Agent / Agentless collection
- Existing CMDB
- Flow log / DNS log
- Application performance monitoring

ToolのInventoryを正と決めつけず、Ownerへの確認と実通信を照合する。

---

# 3. Portfolio Assessment

Workloadごとに評価する。

```text
Business criticality:
Technical complexity:
Migration readiness:
Dependency count:
Data volume:
Downtime tolerance:
Compliance:
License:
Expected benefit:
```

## Candidate分類

- Quick win
- Foundation dependency
- High value / low complexity
- High risk
- Retire candidate
- Modernization candidate
- Retain candidate

最も簡単なApplicationから無条件に始めるのではなく、Landing ZoneやNetworkを検証できる代表Workloadを選ぶ。

---

# 4. 7Rs

## Retire

不要なWorkloadを廃止する。

判断:

- 利用者
- Last access
- Regulatory retention
- Data archive
- Contract

## Retain

現時点では移さない。

理由:

- Hardware dependency
- Regulatory restriction
- Near-term decommission
- Migration benefitが低い

## Rehost

Application変更を最小化し、同等環境へ移す。

候補:

- MGN
- EC2

長所:

- 速い
- Code変更が少ない

短所:

- 既存課題を持ち込む
- Cloud optimizationが限定的

## Relocate

Platform単位で移し、Application変更を抑える。

例:

- VMware環境の移動
- Container platform relocation

## Replatform

一部をManaged化するが、Core architectureは大きく変えない。

例:

- Self-managed DB → RDS
- Web app → Elastic Beanstalk
- Container → ECS

## Repurchase

SaaSや別製品へ置換する。

確認:

- Data export
- Integration
- Identity
- Contract
- User training

## Refactor / Re-architect

Cloud-nativeに再設計する。

例:

- Monolith → services
- Synchronous → event-driven
- Self-managed DB → purpose-built databases

高Valueだが時間・Risk・Skillが必要。

---

# 5. 7R選定フレーム

```text
Change tolerance
  low  → Rehost / Relocate
  medium → Replatform / Repurchase
  high → Refactor

Business value
  low + unused → Retire
  low + constrained → Retain
```

## 強い制約語

| 制約 | 優先候補 |
|---|---|
| 最短期間 | Rehost |
| Code変更不可 | Rehost / Relocate |
| 運用負荷削減 | Replatform / Repurchase |
| Scalability改善 | Replatform / Refactor |
| 製品Support終了 | Repurchase / Refactor |
| 数か月後に廃止 | Retain |

---

# 6. TCO

TCOはAWS料金だけではない。

```text
Current TCO
  = Hardware
  + Data center
  + Power / cooling
  + Network
  + License
  + Operations labor
  + Backup / DR
  + Downtime risk

Target TCO
  = AWS services
  + Data transfer
  + Support
  + License
  + Operations
  + Migration
  + Training
  + Parallel run
```

## One-time cost

- Assessment
- Tool
- Data transfer
- Refactor
- Testing
- Training
- Dual operation
- Contract termination

## Avoided cost

- Hardware refresh
- Data center renewal
- License renewal
- End-of-support risk

短期のMigration費用と長期のRun costを分ける。

---

# 7. Wave Planning

Waveは、Applicationを依存関係とRiskでまとめた移行単位。

## Waveの入力

- Dependency
- Business calendar
- Owner availability
- Data volume
- Downtime window
- Migration strategy
- Target readiness
- Rollback complexity

## 例

```text
Wave 0: Foundation / pilot
Wave 1: Low-risk standalone
Wave 2: Shared internal apps
Wave 3: Core business systems
Wave 4: Complex legacy / modernization
```

## Wave size

大き過ぎる:

- Incident scope拡大
- Team capacity不足
- Rollback困難

小さ過ぎる:

- Coordination overhead
- Project長期化
- Parallel environment cost

Teamが一つのCutover windowで安全に扱える数へ調整する。

---

# 8. Foundation readiness

Workload移行前に整える。

- Organizations / Account structure
- Control Tower / Landing Zone
- IAM Identity Center
- Network connectivity
- DNS
- Logging
- Security services
- KMS
- Backup
- Tagging
- Cost allocation
- Service quotas
- IaC

Workload Teamが毎回独自にAccount、Network、Loggingを作らない。

---

# 9. Network and DNS readiness

```text
On-prem
  ↔ DX / VPN
  ↔ Transit / VPC
  ↔ Target workload
```

確認:

- CIDR overlap
- Route
- BGP
- Bandwidth
- Firewall
- Proxy
- MTU
- DNS inbound / outbound
- Certificate
- Time synchronization

Migration replication trafficと本番trafficを同じ回線へ流す場合、Capacityを計算する。

---

# 10. Identity readiness

- Employee access
- Application authentication
- Workload role
- Directory
- Service account
- Secret
- Certificate

## 人

IAM Identity Center + IdP integration。

## Windows / Legacy

Directory Service、AD Connector、Managed Microsoft AD等を要件に合わせて検討する。

## Workload

Long-term keyをIAM Roleへ置き換える。

移行中はOn-premとAWSでIdentity sourceが二重化しやすい。Authorityを明確にする。

---

# 11. Server Migration with MGN

MGNはSource serverのBlock-level dataを継続複製し、AWS上でTest / Cutover instanceを起動するRehost支援Service。

```text
Source server
  → replication agent
  → staging area
  → test launch
  → validation
  → cutover launch
```

## 適する

- Server rehost
- Short downtime
- Continuous replication
- Physical / virtual source

## 設計

- Staging subnet
- Replication server
- Bandwidth
- Encryption
- Security Group
- Launch template
- Post-launch actions
- Test isolation

## 誤解

MGNはApplication dependency、Data consistency、Business validationを自動的に保証しない。

---

# 12. Database Migration with DMS

DMSはDatabase dataの移行とCDCを支援する。

## Modes

- Full load
- Full load + CDC
- CDC only

```text
Source DB
  → DMS replication instance / serverless
  → Target DB
```

## 同種移行

Native toolとDMSを比較する。

## 異種移行

Schema conversionが必要。

- DMS Schema Conversion
- SCT

Stored procedure、Function、Vendor-specific featureはManual対応が必要な場合がある。

## Cutover

1. Full load完了
2. CDC lag監視
3. Application write停止
4. Lag zero確認
5. Targetへ接続切替
6. Validation
7. Source保持

---

# 13. Database Validation

- Row count
- Checksum
- Key sample
- Referential integrity
- Sequence
- Encoding
- Time zone
- LOB
- Application query
- Performance

Dataが移っただけで、Applicationが同じ結果を返すとは限らない。

---

# 14. File Migration with DataSync

DataSyncはOnline file / object transferを自動化する。

代表:

- NFS
- SMB
- S3
- EFS
- FSx

## 特徴

- Initial copy
- Incremental transfer
- Scheduling
- Verification
- Encryption

## 適する

- File share移行
- Online repeated sync
- Cutover前差分

DB transaction migrationの代替ではない。

---

# 15. Snow Family

Networkで期限内に送れない大量DataをOffline transfer。

概算:

```text
Transfer time = Data size ÷ effective bandwidth
```

Effective bandwidthは公称より低い。

確認:

- Device shipping
- Import / export
- Encryption key
- Chain of custody
- Initial copy後のDelta
- Cutover

Snowで初期Dataを運び、DataSync等で差分を送る組み合わせもある。

---

# 16. Transfer Family

既存ClientがSFTP、FTPS、FTP、AS2などを必要とする場合、Managed endpointでS3 / EFSへ接続する。

用途:

- Partner file exchange
- Protocol維持
- Server運用削減

DataSyncとは目的が違う。

- DataSync: Migration / sync engine
- Transfer Family: Client-facing managed transfer endpoint

---

# 17. S3 Transfer Acceleration

遠距離ClientからS3へInternet uploadするとき、Edgeを経由してAWS networkを利用する。

確認:

- Geography
- Object size
- Existing network
- Additional cost
- Actual speed test

On-prem大容量継続移行ではDX、DataSync、Snowと比較する。

---

# 18. Storage Gateway

MigrationだけでなくHybrid operationに使う。

- File Gateway
- Volume Gateway
- Tape Gateway

「On-premからAWS Storageを継続利用」が要件なら候補。単発CopyだけならDataSync等と比較する。

---

# 19. Cutover Plan

```text
T-30d: rehearsal
T-7d: change freeze確認
T-1d: final readiness
T-0: write stop / drain
  → final sync
  → switch DNS / connection
  → smoke test
  → business validation
  → monitor
```

## 必須項目

- Owner
- Start / end
- Communication
- Freeze
- Data sync
- DNS TTL
- Connection string
- Validation
- Abort condition
- Rollback deadline
- Source shutdown timing

---

# 20. Rollback Plan

Rollbackは「元に戻す」と書くだけでは不十分。

```text
Trigger:
Decision owner:
Latest rollback time:
Traffic return:
Data written after cutover:
Source reactivation:
DNS cache:
User communication:
Validation:
```

## Data divergence

Cutover後にTargetへWriteしたDataがある場合、SourceへTrafficを戻すとDataが失われる可能性がある。

対策:

- Rollback windowを短く定義
- Dual writeのRisk評価
- Reverse replication
- Transaction log handling
- Roll-forward選択

---

# 21. DNS Cutover

- TTLを事前に下げる
- Resolver / JVM cacheを確認
- Weighted routingで段階移行
- Old endpoint trafficを監視
- Certificate hostnameを確認

DNS切替後もExisting connectionは旧環境へ残る場合がある。

---

# 22. Application Validation

## Smoke test

主要機能が起動する。

## Functional test

Use caseが正しく完了する。

## Integration test

External / internal dependency。

## Performance test

Production load。

## Security test

Access、Logging、Encryption。

## Business validation

Order、Payment、Report等の成果。

Health check成功だけでCutover完了にしない。

---

# 23. Hypercare

Cutover直後の集中監視期間。

- Dashboard
- Incident channel
- Owner roster
- Enhanced logging
- Known issue
- Rollback threshold
- Daily review

Hypercare終了条件:

- Error normal
- Performance SLO
- Backup success
- Monitoring complete
- Support handoff
- Runbook updated

---

# 24. Decommission

Sourceをすぐ削除しない。

確認:

- Retention period
- Rollback period
- Data archive
- License termination
- DNS record
- Monitoring
- CMDB
- Backup
- Security credential
- Contract

削除後もSnapshotやLogが残りCostになる可能性がある。

---

# 25. Modernizationの入口

Migration直後に全てRefactorしない。まず安定化し、Evidenceから改善する。

## Candidate

- Manual scaling
- Single point of failure
- Long deployment
- Shared database coupling
- Synchronous dependency
- High operations effort
- License cost
- Poor observability

---

# 26. Monolithの段階的Modernization

Strangler Fig:

```text
Client
  → Routing layer
      ├─ legacy function
      └─ new service
```

機能単位で移す。

## 手順

1. Domain boundary
2. Routing facade
3. New implementation
4. Data ownership
5. Traffic shift
6. Legacy removal

一度に全面RewriteするRiskを減らす。

---

# 27. Decoupling

## Before

```text
Order API
  → Payment
  → Inventory
  → Email
```

一つの遅延が全体へ伝播。

## After

```text
Order API
  → EventBridge / SNS / SQS
      → independent consumers
```

Tradeoff:

- Eventual consistency
- Duplicate
- Ordering
- Observability
- Compensation

---

# 28. Serverless Modernization

候補:

- Lambda
- API Gateway
- EventBridge
- Step Functions
- DynamoDB
- S3

適する:

- Event-driven
- Variable load
- Short execution
- Managed operations

適さない / 比較:

- Long running
- Specialized runtime
- Constant high load
- Strict local state

Serverless化自体を目的にしない。

---

# 29. Container Modernization

## ECS

AWS-nativeで比較的Simple。

## EKS

Kubernetes API、ecosystem、existing manifests。

## Fargate

Node management削減。

## EC2 capacity

Special hardware、daemon、Cost optimization、fine control。

選定軸:

- Kubernetes requirement
- Team skill
- Operational responsibility
- Workload constraint
- Cost

---

# 30. Purpose-built Database

Self-managed relational DBを無条件にDynamoDBへ変えない。

見る:

- Data model
- Access pattern
- Transaction
- Query flexibility
- Consistency
- Scale
- Migration complexity

候補:

- Aurora / RDS
- DynamoDB
- ElastiCache
- Neptune
- OpenSearch
- Redshift
- Timestream

---

# 31. Replatform Case

## Current

Tomcat + self-managed MySQL on VMs。

## Candidate

- Tomcat container on ECS / Elastic Beanstalk
- MySQL on RDS / Aurora
- S3 + CloudFront for static content
- ElastiCache for session

## Benefit

- OS / DB operation削減
- Horizontal scaling
- Managed backup

## Risk

- Session externalization
- DB compatibility
- File system dependency
- Deployment change

---

# 32. Migration Security

- Encryption in transit
- Encryption at rest
- Least privilege role
- Temporary credentials
- Source credential rotation
- Tool logging
- Staging subnet isolation
- Sensitive data masking
- KMS key access
- Agent supply chain

Migration用の一時的なWide permissionを残さない。

---

# 33. Migration Governance

- Account vending
- Region restriction
- Naming
- Tag
- Logging
- Security baseline
- Backup
- Network pattern
- Exception process

Landing Zoneが整う前に大量Workloadを移すと、後から統制を追加するCostが増える。

---

# 34. Migration Program Metrics

## Delivery

- Applications assessed
- Applications migrated
- Wave completion
- Schedule variance
- Cutover success

## Quality

- Incident after migration
- Rollback
- Data reconciliation
- SLO compliance

## Value

- Data center avoided cost
- Operational hours reduced
- Deployment frequency
- Availability
- Unit cost

Server数だけを成功指標にしない。

---

# 35. Common Failure

- Inventoryが不完全
- Owner不明
- Dependency見落とし
- DNSをCutover当日に初確認
- Test環境がProductionと異なる
- Data validation不足
- Rollback時のDataを未設計
- Sourceを早期削除
- Landing Zone未整備
- Rehost後にOptimization計画なし
- 大規模Big Bang
- Business validationなし

---

# 36. SAP-C02の判断

## Code変更最小、短期

MGN + Rehost。

## Database異種移行、停止短縮

Schema Conversion + DMS Full load and CDC。

## NFS / SMBをOnline同期

DataSync。

## Networkで期限に間に合わないPB級Data

Snow Family + Delta sync。

## PartnerがSFTPを維持

Transfer Family。

## Portfolio全体を追跡

Migration Hub + Assessment / Wave plan。

## Operational overhead削減

Replatform / Repurchase。

## Legacyを段階的に分解

Strangler Fig + Integration services。

---

# 37. 誤答パターン

- DataSyncでDatabase CDCを行う
- DMSでServer OSを移す
- MGNでStored procedureを変換する
- Snow deviceで継続差分同期する
- Transfer Familyを一括Migration engineとする
- 7RをTechnologyだけで選ぶ
- DNS変更だけでData consistencyも切り替わる
- Test launch成功だけでBusiness validation完了
- RehostをModernization完了とする
- Rollback時のTarget writeを無視する

---

# 38. Workload Assessment Template

```text
Application:
Business owner:
Criticality:
Users:
Dependencies:
Data stores:
Data volume / growth:
Traffic:
RTO / RPO:
Downtime window:
Compliance:
License:
Current cost:
Operational pain:
7R:
Target architecture:
Migration tools:
Wave:
Test plan:
Cutover:
Rollback:
Modernization backlog:
```

# 39. Cutover Readiness Checklist

- Replication healthy
- Lag within threshold
- Target capacity tested
- Security approved
- Backup taken
- DNS TTL lowered
- Monitoring enabled
- Runbook ready
- Communication sent
- Owner available
- Rollback tested
- Business test prepared
- Source freeze confirmed

# 40. 一文の完成形

> このJava ApplicationはCode変更を最小化し、停止を30分以内にする必要があるため、最初はMGNでRehostする。DatabaseはAurora互換性を評価し、DMSのFull load + CDCで差分を同期する。Cutover時はWriteを停止し、CDC lagとData validationを確認して接続先を切り替える。TargetへWrite開始後のRollback方法を事前に決め、安定化後にSession外出しとECS化を別Waveで行う。

この説明ができれば、Migration toolの暗記ではなく、ProgramとWorkloadの移行を設計できている。

## 関連資料

- [Migration Hub](services/migration/migration-hub.md)
- [Application Discovery Service](services/migration/application-discovery-service.md)
- [Application Migration Service](services/migration/mgn.md)
- [Database Migration Service](services/migration/dms.md)
- [Schema Conversion](services/migration/sct.md)
- [DataSync](services/migration/datasync.md)
- [Snow Family](services/migration/snow-family.md)
- [Transfer Family](services/migration/transfer-family.md)
- [Elastic Disaster Recovery](services/migration/elastic-disaster-recovery.md)
- [Hybrid DNS](comparisons/hybrid-dns-deep-dive.md)
