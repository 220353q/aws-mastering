# Pattern: Multi-Region Active-Active / Active-Passive

## Architecture
Primary Region (Active) ↔ Secondary Region (Standby/Active)
- Route 53 latency-based or failover routing
- DynamoDB Global Tables / Aurora Global Database
- S3 Cross-Region Replication
- CloudFront + Route 53
- Lambda@Edge or Global Accelerator

## Use Cases
- Disaster Recovery (RPO/RTO low)
- Global low latency (active-active)
- Compliance (data residency)

**SAP Focus**: Design for business continuity with <15min RTO, <5min RPO using global services.