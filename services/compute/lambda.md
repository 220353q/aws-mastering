# AWS Lambda

## Overview
AWS Lambda is a serverless compute service that runs code in response to events. No server management, automatic scaling, pay-per-request.

## Key Features
- Runtimes (Python, Node.js, Java, Go, .NET, Ruby, custom)
- Event Sources (API Gateway, S3, DynamoDB, EventBridge, SQS, SNS, Kinesis)
- Layers, Versions, Aliases
- Provisioned Concurrency
- SnapStart (Java)
- Lambda@Edge

## Use Cases (Tier 1 - Selected 5)
1. **Serverless API Backend** - API Gateway + Lambda + DynamoDB (most common SAP pattern)
2. **Event-Driven Data Processing** - S3 upload triggers Lambda for ETL
3. **Real-time Stream Processing** - Kinesis + Lambda for log analysis
4. **Chatbot / Alexa Skills** - Lex + Lambda for conversational AI
5. **Scheduled Tasks** - EventBridge scheduled + Lambda for cron jobs

## Connections
- **API Gateway** : REST/HTTP APIs
- **DynamoDB** : NoSQL backend
- **EventBridge** : Event routing
- **Step Functions** : Orchestration
- **S3** : Object triggers
- **CloudWatch Logs** : Logging

## Comparison
See comparisons/ec2-vs-lambda-fargate.md

## Well-Architected
- Cost: Pay per ms, no idle
- Scalability: Automatic
- Security: IAM execution role, VPC endpoints

**SAP-C02 Focus**: Serverless design principles, event-driven architectures.