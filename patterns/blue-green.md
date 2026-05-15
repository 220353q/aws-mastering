# Pattern: Blue/Green & Canary Deployments

## Architecture
Blue (live) + Green (new version) environments. Route 53 / Global Accelerator / ALB weighted routing for gradual shift. CloudFormation / CDK for IaC swap.

## Key Services
- **Route 53 / Global Accelerator** (weighted / latency routing)
- **Elastic Load Balancing** (target group swap)
- **AWS CodeDeploy** (blue/green deployment config)
- **CloudFormation / CDK** (environment swap)
- **CloudWatch Alarms** (rollback triggers)

## Benefits
- Zero-downtime deployments
- Instant rollback
- Safe production validation

**SAP-C02**: Use for critical workloads; combine with canary (weighted 5-10%) + automated rollback on error rate.