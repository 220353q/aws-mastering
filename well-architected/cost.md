# Well-Architected Cost Optimization Pillar

## Design Principles
- Implement cloud financial management (FinOps culture)
- Adopt a consumption model (pay for what you use)
- Measure overall efficiency (unit economics)
- Stop spending money on undifferentiated heavy lifting (serverless, managed)
- Analyze and attribute expenditure (tagging + Cost Explorer)
- Use managed services to reduce cost (Lambda, DynamoDB on-demand, Aurora Serverless)

## Key Services & Patterns
- Cost Explorer + Budgets + Cost and Usage Report
- Savings Plans + Spot Instances + Reserved
- S3 Intelligent-Tiering + Lifecycle
- Lambda + DynamoDB on-demand + Step Functions Express
- Right-sizing with Compute Optimizer + Trusted Advisor
- Tagging strategy + Organizations consolidated billing

## SAP-C02 Focus Areas
- Cost modeling for different architectures
- Savings Plans vs Reserved vs Spot
- Serverless cost optimization
- Data transfer cost awareness (VPC endpoints, CloudFront, Global Accelerator)

## Week 5: Decision Flow

1. **Observe**: Cost Explorer / CUR / Tags / Cost Categoriesで支出を把握する。
2. **Right-size**: Compute OptimizerでEC2/EBS/Lambda/ECS/RDSなどの過剰・不足を確認する。
3. **Architect**: Auto Scaling、Serverless、S3 Lifecycle、VPC Endpoints、CloudFrontで構造的に下げる。
4. **Commit**: 安定ベースラインにSavings Plans / RIを適用する。
5. **Govern**: Budgetsで予算、利用率、カバレッジを監視する。

## High-Risk Trap

Savings PlansやReserved Instancesは、容量や設計の過剰を直すものではない。SAP-C02では、まず需要パターンと利用実績を分析し、Rightsizing後にコミットメントを検討する流れが安全。
