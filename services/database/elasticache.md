# Amazon ElastiCache

## Overview
Fully managed in-memory cache (Redis or Memcached). Sub-millisecond latency for read-heavy workloads. Supports clustering, replication, encryption.

## Key Features
- Redis (clustering, persistence, pub/sub, Lua scripting) or Memcached
- Cluster mode, Multi-AZ with automatic failover
- Encryption at rest/transit, IAM auth, VPC
- Global Datastore (cross-region read replicas)
- Serverless (new) + Reserved nodes

## Use Cases (Tier 1)
1. **Read-Heavy Web Apps** - Cache RDS/Aurora queries (common SAP pattern)
2. **Session Store** - User sessions with high concurrency
3. **Real-time Leaderboards / Gaming** - Redis sorted sets + pub/sub
4. **Rate Limiting / API Throttling** - Token bucket with Redis
5. **Message Queuing / Pub-Sub** - Redis streams or pub/sub

## Connections
- **RDS / Aurora / DynamoDB**: Backend cache layer
- **EC2 / Lambda / ECS**: Application clients
- **CloudWatch**: Metrics + slow log
- **VPC**: Secure deployment

## Comparison
See comparisons/rds-vs-dynamodb (cache layer on top)

## Well-Architected
- Performance: Sub-ms latency, clustering
- Cost: Right-size nodes + reserved
- Security: Encryption + IAM + VPC
- Reliability: Multi-AZ + automatic failover

**SAP-C02 Focus**: Design high-performance caching layers for read-heavy and real-time workloads.