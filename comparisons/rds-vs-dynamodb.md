# Comparison: RDS vs DynamoDB

| Aspect | RDS (Aurora) | DynamoDB |
|--------|--------------|----------|
| **Data Model** | Relational (SQL) | NoSQL (key-value, document) |
| **Scaling** | Vertical + read replicas | Horizontal, automatic |
| **Consistency** | Strong (ACID) | Eventual / Strong (configurable) |
| **Query** | Complex SQL joins | Simple key/query, GSI |
| **Cost** | Instance + storage | Per request, storage |
| **SAP Use** | Transactional systems, ERP | User profiles, session, IoT, high throughput |

**Choose RDS for**: Complex queries, transactions, existing SQL apps.
**Choose DynamoDB for**: Massive scale, low latency, serverless.