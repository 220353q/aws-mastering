# SAP-C02 Domain 1: Design Solutions for Organizational Complexity

## Key Topics
- Multi-account strategies (Organizations, OU, SCPs, Control Tower)
- IAM best practices (least privilege, roles, identity federation, permission boundaries)
- Cross-account encryption and data access (KMS key policy + IAM policy + resource policy)
- Networking across accounts (Transit Gateway, PrivateLink, VPC Peering)
- Cost allocation & governance (Budgets, Cost Explorer, Organizations consolidated billing)
- Compliance & audit (CloudTrail, Config, Security Hub, Artifact)

## Common Scenarios
- Design multi-account landing zone for enterprise
- Implement cross-account access with least privilege
- Share encrypted S3/EBS/RDS resources across accounts without breaking KMS permissions
- Design governance for 100+ accounts
- Cost visibility and chargeback model
- Centralize security findings and event routing across accounts

## Recommended Services
Organizations + Control Tower + IAM Identity Center + IAM + KMS + Transit Gateway + PrivateLink + CloudTrail + Config + Security Hub + Cost Explorer + Budgets + EventBridge

## High-Risk Exam Traps
- SCPは権限を付与しない。最大権限を制限するだけ。
- IAMで許可してもKMS key policyが入口を開いていなければSSE-KMSデータを読めない。
- PrivateLinkはVPC全体の相互接続ではなく、サービス単位のプライベート接続。
- Direct Connect GatewayはVPC間トランジットルーターではない。

---

## Governance and Cross-Account Design Notes

- Organizations配下でバックアップを標準化する場合、AWS Backup Policy + Cross-Account Backupを検討する。
- 共有サービスを他アカウントへ公開する場合、VPC Peering/TGWではなくPrivateLinkが最小露出になることがある。

---

## Audit, Baseline Deployment, and Cost Governance Notes

- 監査ログはCloudTrail、設定準拠はConfig、所見集約はSecurity Hubと切り分ける。
- 複数アカウント/複数リージョンへ標準リソースを配布する場合はCloudFormation StackSetsを検討する。
- Cost Explorer / Budgets / Cost Allocation Tags / Cost Categoriesで、組織横断の可視化とチャージバックを設計する。

---

## Practice Links

- [Scenario Set 02](../practice/scenario-set-02.md): SCP / Permission Boundary / PrivateLink / CloudFront / Cognito + Lake Formation
