# Pattern: Disaster Recovery (Pilot Light / Warm Standby / Multi-Site)

## Options
- **Pilot Light**: Core services running in DR region (RDS read replica, S3 replication)
- **Warm Standby**: Scaled-down version running, ready to scale
- **Multi-Site Active/Active**: Full production in multiple regions

## Key Services
- **Route 53** (failover routing)
- **DynamoDB Global Tables / Aurora Global**
- **S3 Cross-Region Replication**
- **CloudFormation StackSets** for multi-region IaC
- **AWS Backup** cross-region

**RTO/RPO Targets**:
- Pilot Light: RTO hours, RPO minutes
- Multi-Site: RTO seconds, RPO near-zero

**SAP-C02**: Design DR with business continuity requirements using global AWS services.