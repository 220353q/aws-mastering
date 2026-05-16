# SAP-C02 Domain 4: Accelerate Workload Migration and Modernization

Domain 4 は試験比率としては他ドメインより小さく見えるが、**サービス選択の誤答が起きやすい領域**。サーバー移行、DB移行、ファイル移行、オフライン移行、アプリモダナイゼーションを混同しないことが重要。

---

## Key Topics

- Migration strategies: Rehost, Relocate, Replatform, Repurchase, Refactor/Re-architect, Retain, Retire
- Server migration: [AWS Application Migration Service](../services/migration/mgn.md)
- Database migration: [AWS DMS](../services/migration/dms.md) + [SCT / DMS Schema Conversion](../services/migration/sct.md)
- File/object migration: [AWS DataSync](../services/migration/datasync.md)
- Offline large-scale transfer: [AWS Snow Family](../services/migration/snow-family.md)
- Managed file transfer: [AWS Transfer Family](../services/migration/transfer-family.md)
- Migration tracking and discovery: [Migration Hub](../services/migration/migration-hub.md) + [Application Discovery Service](../services/migration/application-discovery-service.md)
- Modernization patterns: Strangler Fig, Blue/Green, Saga, CQRS, Event Sourcing
- Governance during migration: Organizations + Control Tower + Landing Zone
- Cost & risk management during migration

---

## Service Selection Table

| 要件 | 選ぶサービス |
|---|---|
| サーバーを大きく変更せずEC2へリホスト | MGN |
| DBを最小停止時間で移行 | DMS Full Load + CDC |
| OracleからAurora PostgreSQLなど異種DBへ移行 | SCT / DMS Schema Conversion + DMS |
| NFS/SMBファイルサーバーをS3/EFS/FSxへ移行 | DataSync |
| PB級データを帯域不足環境から移行 | Snow Family |
| 既存SFTP/FTPS/FTP/AS2クライアントを維持 | Transfer Family |
| 大量サーバーの依存関係を把握 | Application Discovery Service |
| 移行ウェーブ全体をアプリ単位で追跡 | Migration Hub |

---

## Common Scenarios

- Design migration strategy for legacy monolith to cloud-native.
- Choose between rehost vs replatform vs refactor.
- Design hybrid connectivity during migration.
- Modernize data platform: data lakehouse + warehouse.
- Implement safe deployment during cutover: blue/green + canary.
- Reduce cutover downtime with replication and pre-testing.
- Discover application dependencies before grouping migration waves.

---

## Recommended Services

Application Migration Service + DMS + SCT / DMS Schema Conversion + DataSync + Snow Family + Transfer Family + Migration Hub + Application Discovery Service + CloudFormation + CDK + Step Functions + EventBridge + Route 53 + Global Accelerator + Strangler Fig + Blue/Green patterns.

---

## Week 4 Additions

- オンプレアプリを変えずNFS/SMB/iSCSI/Tapeを継続するならStorage Gateway。
- 移行後のファイル共有要件は、Linux/NFSならEFS、Windows/SMBならFSx for Windows、HPCならFSx for Lustre。

---

## Week 5 Additions

- 移行先環境の標準展開はCloudFormation StackSetsやControl Towerと組み合わせる。
- 移行後のコスト最適化はCompute Optimizer、Cost Explorer、Savings Plans/RI、Storage Lifecycleで段階的に行う。
- レガシー分析基盤のモダナイズでは、S3 + Glue + Athena + Lake Formation + QuickSight/Quick Sight + DataZoneの役割分担を明確にする。

---

## Practice Links

- [Scenario Set 03](../practice/scenario-set-03.md): MGN / DMS + SCT / DataSync / Snow Family / Transfer Family / DR Cutover
