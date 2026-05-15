# Pattern: Saga (Distributed Transaction)

## Architecture
Orchestrator (Step Functions / Lambda) coordinates local transactions across services with compensating actions on failure.

## Key Services
- **AWS Step Functions** (orchestrator with choice + parallel states)
- **Lambda / ECS** (local transaction participants)
- **DynamoDB / Aurora** (local DB with outbox pattern)
- **EventBridge / SQS** (event publishing)
- **CloudWatch + X-Ray** (distributed tracing)

## Benefits
- No distributed lock / 2PC
- Resilient to partial failures
- Eventual consistency with compensation

**SAP-C02 Example**: Order placement saga (reserve inventory → charge payment → ship → compensate on failure).