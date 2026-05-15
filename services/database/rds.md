# Amazon RDS & Aurora

## Overview
Amazon Relational Database Service (RDS) managed relational DBs. Aurora is MySQL/PostgreSQL compatible with 5x performance, serverless option.

## Key Features
- Engines: MySQL, PostgreSQL, MariaDB, Oracle, SQL Server
- Aurora: Auto-scaling storage, read replicas, global database, serverless
- Multi-AZ, Read Replicas, Snapshots, PITR
- IAM DB Auth, Secrets Manager integration
- Performance Insights, Enhanced Monitoring

## Use Cases (Tier 1)
1. **Transactional Web Apps** - Aurora for high throughput OLTP
2. **ERP/CRM Systems** - Oracle/SQL Server migration
3. **Read-Heavy Apps** - Read replicas + ElastiCache
4. **Serverless DB** - Aurora Serverless v2 for variable workloads
5. **Multi-Region** - Aurora Global Database for low latency global apps

## Connections
- **EC2 / Lambda**: Application backend
- **ElastiCache**: Caching layer
- **S3**: Backups, data import/export
- **CloudWatch**: Monitoring
- **IAM**: Authentication
- **Secrets Manager**: Credential management

## Comparison
See comparisons/rds-vs-dynamodb.md

## Well-Architected
- Reliability: Multi-AZ, PITR, snapshots
- Security: Encryption at rest/transit, IAM DB auth
- Cost: Right-sizing, reserved instances, Aurora serverless
- Performance: Read replicas, Performance Insights

**SAP-C02 Focus**: Design highly available, scalable database architectures with failover and read scaling.