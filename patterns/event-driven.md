# Pattern: Event-Driven Architecture

## Flow
Event Source (S3 upload / DB change / Custom) → EventBridge → Lambda / Step Functions / SQS → Target Services

## Core Services
- **Amazon EventBridge**: Event bus, rules, schema registry
- **AWS Lambda / Step Functions**: Processing / Orchestration
- **Amazon SQS / SNS**: Queuing / Pub-Sub
- **Amazon Kinesis**: Streaming

## Benefits
- Loose coupling
- Scalability
- Auditability (EventBridge)

**SAP Example**: Order placed → EventBridge → Lambda inventory update + SNS notification + Step Functions fulfillment workflow.