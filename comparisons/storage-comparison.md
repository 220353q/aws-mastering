# AWS Storage Comparison

> Storageは容量だけで選ばない。**Access protocol、共有範囲、Latency、Throughput、Durability、Availability、Lifecycle、既存Application互換性**で選ぶ。

## 最初の分類

```text
Object API        → S3
Block device      → EBS / Instance Store
Linux shared file → EFS
Windows / HPC / NAS互換 → FSx
On-prem hybrid    → Storage Gateway
Offline transfer  → Snow Family
Backup governance → AWS Backup
```

---

## 全体比較

| Service | 種類 | Access | Scope | 主用途 |
|---|---|---|---|---|
| S3 | Object | HTTPS API | Region | Static data、Data lake、Backup |
| EBS | Block | EC2 block device | AZ | OS、DB、low-latency disk |
| Instance Store | Local block | Instance local | Host | Temporary high-speed data |
| EFS | File | NFS | Regional / AZ options | Linux shared file |
| FSx for Windows | File | SMB | Managed file system | Windows / AD |
| FSx for Lustre | Parallel file | Lustre | Managed file system | HPC、ML、S3 processing |
| FSx for ONTAP | File / block | NFS、SMB、iSCSI | Managed NAS | NetApp migration、multiprotocol |
| FSx for OpenZFS | File | NFS | Managed file system | ZFS workloads |
| Storage Gateway | Hybrid | NFS/SMB/iSCSI/VTL | On-prem ↔ AWS | Hybrid access、backup |
| S3 Glacier classes | Object archive | S3 API / restore | Region | Long-term archive |
| AWS Backup | Policy plane | Service integrations | Multi-account | Central backup governance |

---

# 1. Object / Block / File

## Object

Object keyでDataを取得する。

```text
bucket + key → object
```

DirectoryやRandom block updateを前提にしない。

## Block

OSからDisk blockとして見える。File systemやDatabaseが上に乗る。

```text
EC2 → EBS volume → file system / database
```

## File

Directory、file name、permissionを使い、複数Clientから共有できる。

```text
Clients → NFS / SMB → shared files
```

---

# 2. Amazon S3

## 選ぶ条件

- Object storage
- High durability
- Static content
- Data lake
- Logs
- Backup
- Event source
- Lifecycle / archive

## 選ばない条件

- POSIX file systemをそのまま要求
- OS boot disk
- Database block device
- In-place file lockingが中心

## 設計論点

- Storage class
- Versioning
- Lifecycle
- Object Lock
- Replication
- Encryption
- Access Point
- CloudFront OAC
- Request cost
- Retrieval time

---

# 3. S3 Storage Class

| Class | Access pattern | Retrieval | 注意 |
|---|---|---|---|
| Standard | Frequent | Immediate | Default general use |
| Intelligent-Tiering | Unknown / changing | Tier依存 | Monitoring / archive tiers |
| Standard-IA | Infrequent、multi-AZ | Immediate | Minimum duration / retrieval charge |
| One Zone-IA | Re-creatable、single AZ | Immediate | AZ loss耐性を要件確認 |
| Glacier Instant Retrieval | Archive but immediate read | Immediate | Minimum duration |
| Glacier Flexible Retrieval | Archive | Minutes〜hours | Restore workflow |
| Glacier Deep Archive | Long-term | Hours | Long RTO |

「安いClass」だけで選ばず、Retrieval、Minimum duration、Requestを含める。

---

# 4. S3 Replication

## Same-Region Replication

同Region別Bucketへ複製。

## Cross-Region Replication

別Regionへ複製。

用途:

- Compliance
- DR
- Data locality
- Account separation

## 注意

- Versioning
- Existing objectsへの適用方法
- Delete marker
- KMS permissions
- Replication lag
- Transfer cost

ReplicationはBackupの完全代替ではない。誤操作や破損が複製され得る。

---

# 5. EBS

## 選ぶ条件

- EC2 boot volume
- Database disk
- Low-latency block I/O
- Snapshot

## Scope

VolumeはAZに属する。別AZ Instanceへ直接Attachする前提にしない。

## Performance

- IOPS
- Throughput
- Volume queue
- Instance EBS bandwidth

## Volume type

| Type | 主用途 |
|---|---|
| gp3 | General purpose、独立IOPS/throughput |
| gp2 | Legacy general purpose |
| io2系 | High IOPS / durability |
| st1 | Large sequential throughput |
| sc1 | Cold throughput workloads |

## Snapshot

Incremental backup。Restore後の初回Access性能とFast Snapshot Restoreを確認する。

---

# 6. EBS Multi-Attach

対応Volume / Instance条件で複数InstanceへAttachできるが、ApplicationとFile systemがConcurrent accessを安全に扱う必要がある。

「共有File systemが必要」だから即Multi-Attachとはしない。EFSやFSxと比較する。

---

# 7. Instance Store

Instance Hostに物理的に付随するTemporary storage。

## 長所

- High performance
- No separate EBS charge in model

## Risk

Stop / terminate / host failureでDataを失う可能性。正本を置かない。

適する:

- Cache
- Scratch
- Replicated distributed data
- Temporary processing

---

# 8. EFS

## 選ぶ条件

- Linux
- NFS
- Multiple clients
- Elastic capacity
- Regional shared file

## 設計

- Mount target per AZ
- Security Group
- Access Point
- Performance mode
- Throughput mode
- Lifecycle to IA / Archive

## 注意

- Small file / metadata heavy workload
- File permission
- Cross-AZ mount path
- Application locking

Windows SMBならFSx for Windowsを検討する。

---

# 9. FSx for Windows File Server

## 選ぶ条件

- SMB
- Windows ACL
- Active Directory
- DFS
- Existing Windows file workloads

## 注意

- Single-AZ / Multi-AZ option
- Throughput capacity
- Storage type
- AD connectivity
- Backup

EFSはNFS中心であり、Windows file server互換の代替と決めつけない。

---

# 10. FSx for Lustre

## 選ぶ条件

- HPC
- ML training
- Parallel processing
- High throughput
- S3 dataset integration

Persistent / scratch等のDeployment特性をWorkloadに合わせる。

一般的なOffice file shareには選ばない。

---

# 11. FSx for ONTAP

## 選ぶ条件

- NetApp feature
- NFS / SMB / iSCSI
- Snapshot / clone
- Tiering
- Existing NAS migration

MultiprotocolやStorage efficiencyが強い要件の場合に有力。

---

# 12. FSx for OpenZFS

## 選ぶ条件

- ZFS / NFS workload
- Snapshot / clone
- Low-latency managed file
- Existing ZFS migration

Protocol互換とFeatureを確認する。

---

# 13. Storage Gateway

## File Gateway

On-premへNFS / SMB interfaceを提供し、DataをS3へObjectとして保存。

## Volume Gateway

iSCSI block volume。Cached / stored modelを要件に合わせる。

## Tape Gateway

Virtual tape libraryとしてBackup softwareと連携。

## 選ぶ条件

- Hybrid operation
- Existing protocol維持
- Local cache
- Gradual cloud adoption

単発TransferだけならDataSyncと比較する。

---

# 14. DataSyncとの違い

| 観点 | Storage Gateway | DataSync |
|---|---|---|
| 主目的 | 継続Hybrid access | Online transfer / sync |
| Interface | NFS/SMB/iSCSI/VTL | Transfer agent / service |
| Local cache | あり得る | 主目的でない |
| Migration | 段階移行 | Initial / incremental copy |

---

# 15. Snow Family

Network transferが期限に間に合わない大量DataをDeviceで移送する。

判断:

```text
Data size ÷ effective bandwidth
  > available migration window
```

Initial loadをSnow、差分をDataSync / networkで送る組み合わせがある。

---

# 16. AWS Backup

AWS BackupはStorageそのものではなく、Backup policyとVaultを集中管理するControl plane。

## 機能

- Backup plan
- Vault
- Cross-account copy
- Cross-Region copy
- Organizations policy
- Vault Lock
- Restore testing

各Service native backupとの役割を理解する。

---

# 17. Backup / Snapshot / Replica

| 用語 | 主目的 | 誤操作からの復旧 |
|---|---|---|
| Snapshot / Backup | 過去時点へ戻す | 可能 |
| Replica | Availability / read scale | 誤操作が伝播する可能性 |
| Multi-AZ standby | Failover | 過去時点復旧ではない |
| Versioning | Object履歴 | 誤削除対策に有効 |

ReplicaをBackupとして扱わない。

---

# 18. Availability Scope

## AZ resource

- EBS
- Instance Store

## Regional service / architecture

- S3
- EFS regional
- Multi-AZ FSx option

Scopeを理解し、AZ障害時の再Attach / Restore / Failoverを設計する。

---

# 19. Performance軸

```text
Small random I/O → IOPS
Large sequential → Throughput
Metadata heavy   → File system behavior
Single request   → Latency
Parallel clients → Aggregate throughput
```

容量だけで性能を推定しない。

---

# 20. Protocol軸

| Requirement | Candidate |
|---|---|
| HTTPS Object API | S3 |
| Block device | EBS |
| Linux NFS | EFS / FSx variants |
| Windows SMB / AD | FSx for Windows |
| Parallel Lustre | FSx for Lustre |
| NetApp multiprotocol | FSx for ONTAP |
| On-prem tape software | Tape Gateway |

---

# 21. Cost軸

見る:

- Stored GB
- Provisioned performance
- Request
- Retrieval
- Minimum duration
- Data transfer
- Backup
- Replication
- Throughput capacity

安いStorage classへ移しても、頻繁なRetrievalで高くなる場合がある。

---

# 22. Lifecycle

Dataを温度で分類する。

```text
Hot
  → Warm
  → Cold
  → Archive
  → Delete
```

記録:

- Last access
- Legal retention
- Restore RTO
- Re-creation possible
- Sensitivity

---

# 23. Encryption

- S3 SSE-S3 / SSE-KMS等
- EBS encryption
- EFS encryption
- FSx encryption
- Backup encryption

Cross-account / Cross-RegionではKMS Key policyとGrantを確認する。

EncryptionはAccess authorizationの代替ではない。

---

# 24. Migration選択

| Source / requirement | Service |
|---|---|
| NFS / SMB online sync | DataSync |
| PB-scale offline | Snow Family |
| Existing SFTP clients | Transfer Family |
| Ongoing hybrid NFS/SMB | File Gateway |
| VMware block migration | MGN for servers / relevant tools |
| Database records | DMS |

Storage Dataだから全てDataSyncとは限らない。Data modelとProtocolを見る。

---

# 25. SAP-C02問題文の読み替え

## Static website assets

S3 + CloudFront。

## Shared Linux home directory

EFS。

## Windows file share with AD

FSx for Windows。

## HPC reads S3 dataset in parallel

FSx for Lustre。

## Low-latency database disk on EC2

EBS gp3 / io2系を要件で選択。

## On-prem backup software needs virtual tape

Tape Gateway。

## Central organization backup

AWS Backup + cross-account Vault。

## Immutable archive

S3 Object Lock / Glacier classとRetention要件。

---

# 26. よくある誤答

- S3をPOSIX file systemとしてそのまま利用
- EBSを別AZから直接共有
- EFSをWindows SMB要件へ選ぶ
- Multi-AttachならFile locking不要
- ReplicaをBackupとみなす
- Glacier Deep Archiveを即時復旧要件へ選ぶ
- One Zone-IAを唯一の正本へ使い、AZ lossを無視
- Storage GatewayとDataSyncを同一目的として扱う
- SnapshotがあるからRestore test不要
- Volume IOPSだけ上げてEC2 EBS bandwidthを見ない

---

# 27. 設計テンプレート

```text
Data type:
Protocol:
Clients:
Shared or single host:
Read / write pattern:
Random / sequential:
Latency:
IOPS:
Throughput:
Capacity / growth:
AZ / Region scope:
Durability:
RTO / RPO:
Lifecycle:
Encryption:
Backup:
Migration source:
Cost drivers:
```

## 完成した説明

> 複数AZのLinux ECS Taskが同じDirectoryをNFSで共有し、容量が予測不能に増えるためEFSを採用する。各AZにMount Targetを配置し、Access PointでApplicationごとのPathと権限を分離する。EBSはAZ単位のBlock deviceで共有File要件に合わず、S3はObject APIのため既存POSIX accessを変更しない条件を満たさない。

## 関連資料

- [S3](../services/storage/s3.md)
- [EBS](../services/storage/ebs.md)
- [EFS](../services/storage/efs.md)
- [FSx](../services/storage/fsx.md)
- [Storage Gateway](../services/storage/storage-gateway.md)
- [S3 Glacier](../services/storage/s3-glacier.md)
- [AWS Backup](../services/storage/backup.md)
- [DataSync](../services/migration/datasync.md)
- [Snow Family](../services/migration/snow-family.md)
