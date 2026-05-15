# Comparison: DynamoDB vs Aurora (MySQL/PostgreSQL)

| Aspect | DynamoDB | Aurora (MySQL/PostgreSQL) |
|--------|----------|---------------------------|
| **Data Model** | NoSQL (key-value/document) | Relational (tables, SQL, joins) |
| **Scaling** | Automatic horizontal (unlimited) | Vertical + read replicas (up to 15) |
| **Latency** | Single-digit ms (consistent) | Low ms (with DAX/ElastiCache) |
| **Query Power** | Simple key/GSI queries | Full SQL + complex joins, transactions |
| **Transactions** | ACID transactions (limited) | Full ACID, MVCC |
| **Cost** | Per request + storage | Instance hours + storage + I/O |
| **Best For** | High throughput, simple access, global scale | Complex queries, existing SQL apps, strong consistency |
| **SAP Example** | User profiles, carts, IoT telemetry | ERP, financial transactions, reporting |

**Recommendation**: DynamoDB for scale-first, serverless, simple access patterns. Aurora for complex relational workloads or SQL migration.