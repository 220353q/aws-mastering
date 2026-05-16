# SAP-C02 長文シナリオ読解フレーム

SAP-C02はサービス名の暗記ではなく、制約条件の読み取りとサービス選定理由の説明が問われる。長文問題では、本文中に散らされた要件を次の順に回収する。

---

## 1. まず「制約語」を拾う

| 制約語 | 読み替え | 典型サービス/論点 |
|---|---|---|
| 複数アカウント / 組織全体 | Organizations, SCP, 委任管理, 中央監査 | IAM, SCP, CloudTrail, Config, Security Hub |
| 最小権限 / 委任 | Permission Boundary, ABAC, AssumeRole | IAM, KMS, Resource Policy |
| インターネット公開したくない | PrivateLink, VPC Endpoint, Direct Connect/VPN | PrivateLink, Interface Endpoint |
| 既存環境を大きく変えない | Rehost, Replatform | MGN, DMS, DataSync |
| 停止時間を短く | CDC, Blue/Green, Global DB, Replication | DMS, Aurora Global Database, Route 53 |
| RTO/RPO | DR戦略の選定 | Pilot Light, Warm Standby, Multi-Site |
| 複数購読者 / 非同期 / DLQ | 疎結合イベント設計 | EventBridge, SNS, SQS |
| 列レベル / データ利用申請 | Data Lake Governance | Lake Formation, DataZone |
| コスト削減 | 可視化→Rightsizing→割引 | Cost Explorer, Compute Optimizer, Savings Plans |

---

## 2. サービス名に飛びつかず「対象」を確認する

```text
サーバー移行？        → MGN
DB移行？              → DMS + SCT / DMS Schema Conversion
ファイル移行？        → DataSync
オフライン大容量？    → Snow Family
SFTP/FTPS/FTP維持？   → Transfer Family
オンプレからAWSストレージ継続利用？ → Storage Gateway
```

この対象の切り分けを間違えると、SAP-C02では魅力的な誤答に落ちやすい。

---

## 3. 「権限」は3層で読む

```text
誰が？        Principal / Role / User / Service
何に？        Resource / Data / KMS key / API
どの上限で？  SCP / Permission Boundary / Session Policy / Resource Policy
```

特にSSE-KMS、S3、Lake Formation、クロスアカウントが絡む場合は、IAMだけでなくKMS key policyやLake Formation権限も確認する。

---

## 4. 「通信」は公開範囲で読む

```text
Public HTTP配信              → CloudFront / ALB / WAF / Shield
グローバル固定IP/高速切替     → Global Accelerator
VPC間フル接続                → Transit Gateway / Peering
サービス単位のプライベート公開 → PrivateLink
オンプレ専用接続              → Direct Connect
暗号化トンネル                → Site-to-Site VPN
```

PrivateLinkを「VPC全体の相互接続」と誤読しない。Direct Connectを「暗号化済み通信」と自動解釈しない。

---

## 5. 「DR」はRTO/RPOとコストで読む

| 戦略 | RTO/RPO | コスト | 典型判断 |
|---|---|---|---|
| Backup & Restore | 長い | 低 | 非重要・低コスト |
| Pilot Light | 中 | 低〜中 | コアのみ常時準備 |
| Warm Standby | 短め | 中〜高 | 縮小版を常時稼働 |
| Multi-Site Active/Active | 最短 | 高 | 常時両系稼働・高可用 |

Aurora Global Databaseは、基本的にPrimary Regionへ書き込み、Secondary Regionは読み取り/DR用途として読む。一般的なmulti-writerだと決めつけない。

---

## 6. 誤答の典型パターン

- 監査ログをCloudWatchだけで代替する
- ConfigとCloudTrailを混同する
- GuardDutyをコンプライアンス評価サービスとして扱う
- KMS暗号化を列レベル権限の代替にする
- Cognito MFAをLake Formation権限の自動伝播とみなす
- Snow Familyを継続差分同期サービスとして扱う
- DataSyncをDB移行サービスとして扱う
- Savings PlansをRightsizingの代替にする
- CloudFrontをVPC間プライベート接続やDB高速化に使う
- SCPを権限付与サービスとして扱う

---

## 7. 解答時の一文テンプレート

迷ったら、選択肢を次の文型で検証する。

```text
この選択肢は、[対象] に対して [サービス] を使っており、
[制約条件] を満たす / 満たさない。
なぜなら、[サービスの本来の役割] は [役割] であり、
[本文中の要件] には [別サービス/別設計] が必要だから。
```

このテンプレートで説明できない選択肢は、暗記で選んでいる可能性が高い。
