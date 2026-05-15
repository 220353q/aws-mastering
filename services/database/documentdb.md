# Amazon DocumentDB (with MongoDB compatibility)

## Overview
Fully managed document database compatible with MongoDB 3.6/4.0/5.0. Same MongoDB API, drivers, and tools. 3x better performance than standard MongoDB.

## Key Features
- MongoDB 3.6/4.0/5.0 compatible (API, drivers, tools)
- 3x performance vs standard MongoDB
- Up to 15 read replicas
- Global clusters (cross-region)
- Encryption, IAM, VPC, audit logs
- Change streams, transactions (4.0+)
- Storage auto-scaling

## Use Cases (Tier 2)
1. **Content Management / Catalogs** - JSON documents with flexible schema
2. **User Profiles & Preferences** - High read/write with flexible attributes
3. **IoT / Mobile Backend** - Document storage with MongoDB ecosystem
4. **Real-time Analytics** - Change streams + Lambda
5. **MongoDB Migration** - Lift-and-shift with minimal changes + 3x performance

## Connections
- **Lambda / ECS / EKS**: Application backend
- **EventBridge**: Change stream events
- **S3**: Bulk load / export
- **CloudWatch**: Monitoring
- **IAM**: Authentication

## Well-Architected
- Performance: 3x faster than MongoDB
- Reliability: Multi-AZ + Global clusters
- Security: Encryption + IAM + VPC
- Cost: Instance + storage (auto-scaling)

**SAP-C02**: Design document-oriented solutions with MongoDB compatibility and managed operations.