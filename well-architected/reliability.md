# Well-Architected Reliability Pillar

## Design Principles
- Automatically recover from failure (Auto Scaling, Multi-AZ)
- Test recovery procedures (Chaos Engineering, FIS)
- Scale horizontally to increase availability (stateless, load balancing)
- Stop guessing capacity (right-size + Auto Scaling)
- Manage change in automation (IaC, pipelines)

## Key Services & Patterns
- Multi-AZ + Auto Scaling + ELB
- Route 53 failover + Global Accelerator
- DynamoDB Global Tables + Aurora Global
- S3 Cross-Region Replication + Backup
- Step Functions + EventBridge for resilient workflows
- FIS (Fault Injection Simulator) for chaos testing

## SAP-C02 Focus Areas
- RTO/RPO targets with DR patterns
- Chaos engineering with FIS
- Stateless design + horizontal scaling
- Automated failover and recovery