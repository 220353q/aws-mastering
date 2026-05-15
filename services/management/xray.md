# AWS X-Ray

## Overview
Distributed tracing service. Analyze and debug distributed applications with end-to-end request tracing, service maps, and performance insights.

## Key Features
- End-to-end request tracing (trace ID across services)
- Service map visualization
- Trace analytics, annotations, metadata
- Sampling rules + integration with CloudWatch
- Lambda, ECS, EKS, EC2, API Gateway, Step Functions support
- Insights (anomaly detection in traces)

## Use Cases (Tier 2)
1. **Debug Latency in Microservices** - Trace requests across Lambda, DynamoDB, RDS
2. **Performance Optimization** - Identify slow services / dependencies
3. **Error Root Cause Analysis** - See full request path with errors
4. **Service Dependency Mapping** - Understand architecture in production
5. **SLA Monitoring** - Trace-based performance metrics

## Connections
- **CloudWatch + CloudWatch Logs**: Metrics + logs correlation
- **Lambda / ECS / EKS / Step Functions**: Instrumentation
- **API Gateway**: Tracing from edge
- **EventBridge**: Trace context propagation

## Well-Architected
- Performance: Identify bottlenecks
- Operational Excellence: Root cause analysis
- Reliability: Understand failure propagation

**SAP-C02 Focus**: Design observable distributed systems with end-to-end tracing.