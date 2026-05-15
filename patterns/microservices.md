# Pattern: Microservices Architecture

## Architecture
API Gateway → Lambda / ECS Fargate (per service) → DynamoDB / Aurora (per service) + EventBridge (inter-service comms)

## Key Services
- **Amazon API Gateway** + **AWS Lambda** / **Amazon ECS Fargate**
- **Amazon EventBridge** or **Amazon MQ** for async
- **Amazon DynamoDB** / **Aurora** per service (polyglot persistence)
- **AWS X-Ray** + **Amazon CloudWatch** for observability
- **Amazon Cognito** + **AWS WAF** for security

## Benefits
- Independent deployment & scaling
- Technology diversity
- Fault isolation

**SAP-C02**: Use for complex domains; start with Strangler Fig pattern for monolith migration.