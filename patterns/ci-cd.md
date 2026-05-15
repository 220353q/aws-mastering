# Pattern: CI/CD Pipeline

## Architecture
CodeCommit / GitHub → CodePipeline (Source) → CodeBuild (Build/Test) → CodeDeploy / ECS Deploy → CloudFormation / CDK (IaC)

## Key Services
- **AWS CodePipeline, CodeBuild, CodeDeploy, CodeCommit**
- **AWS CloudFormation / CDK / SAM** for IaC
- **Amazon ECR** for container images
- **AWS Systems Manager** for patching
- **Amazon CloudWatch** for pipeline monitoring

## Benefits
- Automated testing & deployment
- IaC for reproducibility
- Rollback capability

**SAP-C02**: Infrastructure as Code + automated pipelines for reliable, auditable deployments.