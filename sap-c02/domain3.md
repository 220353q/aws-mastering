# SAP-C02 Domain 3: Continuous Improvement

## Key Topics
- Well-Architected Framework reviews (tool + process)
- Cost optimization (right-sizing, Savings Plans, serverless, endpoint/NAT cost tradeoffs)
- Performance optimization (caching, read replicas, global tables, CloudFront)
- Operational excellence (IaC, CI/CD, automation, chaos engineering)
- Security & compliance continuous monitoring (GuardDuty + Security Hub + Config + CloudTrail)
- Feedback loops (CloudWatch + X-Ray + post-mortems)
- Event-driven remediation and automated operations (EventBridge + Systems Manager + Lambda)

## Common Scenarios
- Conduct Well-Architected review and remediate high-risk issues
- Optimize cost of existing architecture
- Improve reliability with chaos engineering
- Implement continuous compliance monitoring
- Route operational/security events to automated remediation workflows
- Reduce NAT Gateway usage by replacing AWS service traffic with VPC Endpoints

## Recommended Services
Well-Architected Tool + Trusted Advisor + Compute Optimizer + FIS + GuardDuty + Security Hub + CloudWatch + X-Ray + Systems Manager + EventBridge + Step Functions + Config + CloudTrail + KMS

## High-Risk Exam Traps
- CloudWatchはメトリクス/ログ/アラーム、CloudTrailはAPI監査、Configは設定履歴/準拠評価。
- EventBridgeは検知イベントを自動修復へつなぐルーターとして使えるが、複雑な状態管理はStep Functions。
- コスト最適化では、NAT GatewayをPrivateLink/Gateway Endpointで置き換えられるケースがある。

---

## Operational Control and Secrets Management Notes

- バックアップ統制・監査・クロスアカウント保護はAWS Backupを検討する。
- Secrets Managerはローテーション、Parameter Storeは設定管理寄り。

---

## Cost, Compliance, and Automated Remediation Notes

- コスト改善はCost Explorerで分析し、Compute OptimizerでRightsizingしてからSavings Plans/RIを検討する。
- 継続的コンプライアンスはConfig Rules/Conformance Packs、セキュリティ所見集約はSecurity Hub。
- CloudTrail/Config/Security Hub/EventBridge/Systems Manager Automationで検知から自動修復まで設計できる。

---

## Practice Links

- [Scenario Set 03](../practice/scenario-set-03.md): DR / Cost Optimization
- [Scenario Set 04](../practice/scenario-set-04.md): Continuous Compliance / Automated Remediation
