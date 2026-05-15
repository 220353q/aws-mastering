# Pattern: CQRS (Command Query Responsibility Segregation)

## Architecture
Write Path (Command): API Gateway → Lambda/Command Handler → EventBridge / Kinesis → DynamoDB / Event Store
Read Path (Query): API Gateway → Lambda/Query Handler → DynamoDB GSI / ElastiCache / Redshift

## Key Services
- **API Gateway + Lambda** (separate for read/write)
- **EventBridge / Kinesis** (event sourcing)
- **DynamoDB** (write model) + GSI / ElastiCache (read model)
- **Step Functions** (saga for complex commands)
- **CloudWatch + X-Ray** (observability)

## Benefits
- Independent scaling of read/write
- Optimized data models per workload
- Event sourcing for audit + replay

**SAP-C02**: Use for high-read or complex domain models; combine with Event Sourcing for full auditability.