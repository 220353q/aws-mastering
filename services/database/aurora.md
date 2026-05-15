# Amazon Aurora

## Overview
MySQL and PostgreSQL compatible relational database with 5x better performance than standard MySQL, auto-scaling storage, and serverless option. Part of RDS but with superior architecture.

## Key Features
- Storage auto-scaling (up to 128 TiB)
- Up to 15 read replicas (low latency)
- Global Database (cross-region read replicas with <1s lag)
- Serverless v2 (auto pause/resume, scaling to millions of requests)
- Backtrack (rewind database without restore)
- Parallel query, Aurora Machine Learning
- Multi-AZ, PITR, snapshots, encryption

## Use Cases (Tier 1 - Selected 5)
1. **High-Throughput OLTP** - E-commerce, SaaS, financial transactions (most common SAP use)
2. **Global Applications** - Aurora Global Database for multi-region low-latency reads
3. **Serverless Workloads** - Aurora Serverless v2 for spiky or unpredictable traffic
4. **Analytics + Transactional** - Parallel query + Redshift Spectrum integration
5. **Legacy MySQL/PostgreSQL Migration** - Minimal code changes, huge performance gain

## Connections
- **EC2 / Lambda / ECS**: Application backend
- **ElastiCache**: Read caching layer
- **S3**: Backups, data import (LOAD DATA FROM S3)
- **CloudWatch + Performance Insights**: Monitoring
- **IAM DB Authentication**: Passwordless access
- **Secrets Manager**: Credential management
- **EventBridge**: Database events

## Comparison
See comparisons/dynamodb-vs-aurora.md and rds-vs-dynamodb.md

## Well-Architected
- Reliability: Multi-AZ, Global Database, Backtrack
- Performance: 5x MySQL, parallel query, serverless
- Cost: Serverless v2 for variable loads, reserved for steady
- Security: Encryption + IAM DB auth + VPC

**SAP-C02 Focus**: Design high-performance, globally distributed relational databases with minimal operational overhead.