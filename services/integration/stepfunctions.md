# AWS Step Functions

## Overview
Serverless workflow orchestration for coordinating multiple AWS services into resilient workflows. Visual state machines.

## Key Features
- Standard vs Express workflows
- States: Task, Choice, Parallel, Map, Wait, Fail, Succeed, Pass
- Error handling, retries, catch
- Callback patterns, Distributed Map
- Express for high-volume / short duration
- Integration with 200+ AWS services

## Use Cases (Tier 1)
1. **Order Fulfillment Workflow** - Lambda → SQS → Step Functions → SNS notification (common e-commerce)
2. **Data Processing Pipeline** - Parallel Map state for batch ETL
3. **Human Approval Workflows** - Callback + Wait states
4. **Multi-Service Orchestration** - Saga pattern for distributed transactions
5. **Scheduled + Event-Driven** - EventBridge → Step Functions

## Connections
- **Lambda / ECS / Batch**: Compute tasks
- **SQS / SNS / EventBridge**: Messaging
- **DynamoDB / RDS**: State persistence
- **CloudWatch**: Execution history + metrics

## Comparison
See comparisons/lambda-vs-stepfunctions.md

## Well-Architected
- Reliability: Built-in retries, error handling, state persistence
- Cost: Express for high volume, Standard for long-running
- Observability: Full execution history
- Security: IAM execution role

**SAP-C02 Focus**: Design resilient, auditable multi-step workflows with error handling and state management.