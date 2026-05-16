# Amazon EFS - Elastic File System

## Positioning

Amazon EFS は、Linux ワークロード向けの **フルマネージド NFS 共有ファイルストレージ**。複数の EC2、ECS、EKS、Lambda から同じファイルシステムを同時にマウントできる。

SAP-C02 では、次のような要件で出る。

- 複数AZのEC2から同じファイルを共有したい
- LinuxアプリケーションがNFS互換の共有ストレージを必要としている
- 容量見積もりを避け、自動的に伸縮するファイルストレージが必要
- コンテナやLambdaから共有ファイルにアクセスしたい
- Windows SMBではなく、Linux/NFS前提である

---

## Core Concepts

| 項目 | 内容 |
|---|---|
| プロトコル | NFS |
| 主な対象 | Linux / POSIX系ワークロード |
| スケール | 容量を自動的に伸縮 |
| 可用性タイプ | Regional / One Zone |
| 接続単位 | VPC内のMount Target |
| アクセス制御 | Security Group + NFS Client + EFS Access Point + IAM認証 |
| 暗号化 | at rest / in transit |

EFS は「ファイル共有」のサービスであり、S3のようなオブジェクトストレージではない。アプリケーションがファイルシステムAPIを前提にしている場合に選ぶ。

---

## Regional vs One Zone

| タイプ | 特徴 | 試験での判断 |
|---|---|---|
| Regional | 複数AZに冗長保存 | 本番・高可用性・DR要件あり |
| One Zone | 単一AZに保存 | 低コスト・再作成可能データ・開発/検証 |

SAP-C02 では、**Multi-AZの可用性を要求されたらRegional EFS** を選ぶ。One Zoneは安価だがAZ障害時のリスクがあるため、本番の重要データでは慎重に扱う。

---

## Performance / Throughput

| 判断軸 | 選択肢 | 向くケース |
|---|---|---|
| Performance Mode | General Purpose | 一般的な低レイテンシ用途 |
| Performance Mode | Max I/O | 大量クライアント・高並列。ただしレイテンシ増加に注意 |
| Throughput Mode | Bursting / Elastic / Provisioned | ワークロード変動、安定高スループット要件で選択 |

試験では細かな数値より、**低レイテンシ一般用途か、大規模並列アクセスか** の切り分けが重要。

---

## Access Point

EFS Access Point は、アプリケーションごとにルートディレクトリ、POSIXユーザー、権限を固定する入口。

### Useful when

- ECS/EKS/Lambdaなど複数アプリが同じEFSを使う
- アプリごとにディレクトリを分離したい
- POSIX権限の誤設定を減らしたい
- IAMと組み合わせてアクセス制御したい

---

## EFS vs S3 vs EBS vs FSx

| 要件 | 選ぶサービス |
|---|---|
| オブジェクト、静的コンテンツ、データレイク | S3 |
| EC2に低レイテンシのブロックストレージ | EBS |
| 複数Linuxホストから共有ファイル | EFS |
| Windows SMB / Lustre / ONTAP / OpenZFS | FSx |

---

## SAP-C02 Focus

EFSを選ぶ典型文脈:

- 「複数のLinux EC2インスタンスが同じファイルを共有」
- 「NFS互換が必要」
- 「スケールする共有ファイルシステム」
- 「ECS/EKS/Lambdaから共有ファイルにアクセス」

EFSを選ばない典型文脈:

- Windows SMBが必要 → FSx for Windows File Server
- HPCで高性能並列ファイルシステム → FSx for Lustre
- オブジェクトストレージで十分 → S3
- 1台のEC2専用のブロックストレージ → EBS

---

## Exam Traps

- EFSはNFS。Windows SMB要件ならFSx for Windows。
- EFS Regionalは複数AZに冗長化されるが、マウントターゲットは各AZに配置する設計が基本。
- EFSはファイルストレージ。S3のようなオブジェクトAPIではない。
- 低コストだけでOne Zoneを選ぶと、本番可用性要件に反することがある。

---

## Related

- [Amazon S3](s3.md)
- [Amazon FSx](fsx.md)
- [AWS Storage Gateway](storage-gateway.md)
- [Storage Options Comparison](../../comparisons/storage-options.md)

## Official Docs

- https://docs.aws.amazon.com/efs/latest/ug/features.html
- https://docs.aws.amazon.com/efs/latest/ug/performance.html
