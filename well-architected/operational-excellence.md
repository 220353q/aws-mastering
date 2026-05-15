# Well-Architected Operational Excellence Pillar

## Design Principles
- Perform operations as code (IaC everywhere)
- Make frequent, small, reversible changes (CI/CD + feature flags)
- Refine operations procedures frequently (runbooks + automation)
- Anticipate failure (chaos engineering + FIS)
- Learn from all operational failures (post-mortems + CloudWatch)

## Key Services & Patterns
- CloudFormation / CDK / SAM / Terraform
- CodePipeline + CodeBuild + CodeDeploy + ECR
- Systems Manager (Automation, Patch Manager, Session Manager)
- FIS (Fault Injection Simulator) + X-Ray + CloudWatch
- EventBridge + Step Functions for automation
- CloudTrail + Config + Security Hub for governance

## SAP-C02 Focus Areas
- IaC best practices
- CI/CD + blue/green/canary
- Automated operations with Systems Manager
- Observability with X-Ray + CloudWatch + OpenSearch