# Pattern: Serverless Web Application

## Architecture
```
Client (Browser/Mobile)
  ↓
API Gateway (REST/HTTP)
  ↓
Lambda (Backend Logic)
  ↓
DynamoDB (Data Store) + S3 (Static Assets)
  ↓
CloudFront (CDN)
  ↓
Route 53 (DNS)
```

## Components
- **Amazon API Gateway**: Frontend API
- **AWS Lambda**: Business logic (Python/Node)
- **Amazon DynamoDB**: NoSQL database
- **Amazon S3 + CloudFront**: Static hosting + CDN
- **Amazon Cognito**: Authentication
- **Amazon EventBridge**: Async events

## Benefits
- Zero server management
- Automatic scaling (0 to millions)
- Pay-per-use
- High availability by default

## SAP-C02 Design Considerations
- Security: Cognito + IAM + API Gateway auth
- Reliability: Multi-region DynamoDB global tables
- Cost: Lambda + DynamoDB on-demand
- Performance: CloudFront caching, DynamoDB DAX

**Implementation**: Use AWS SAM or CDK for IaC.