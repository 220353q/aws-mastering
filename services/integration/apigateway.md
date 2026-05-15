# Amazon API Gateway

## Overview
Fully managed service for creating, publishing, maintaining, monitoring, and securing REST, HTTP, and WebSocket APIs at any scale.

## Key Features
- REST API (with API keys, usage plans, throttling)
- HTTP API (low cost, high performance)
- WebSocket API (real-time bidirectional)
- Lambda / HTTP / Mock / AWS Service integrations
- Authorizers (Cognito, Lambda, IAM)
- Request/Response mapping, validation, caching
- Canary deployments, mutual TLS

## Use Cases (Tier 1)
1. **Serverless REST/HTTP Backend** - API Gateway + Lambda + DynamoDB (core SAP pattern)
2. **Real-time Chat / Notifications** - WebSocket API + Lambda
3. **API Throttling & Monetization** - Usage plans + API keys
4. **Legacy Modernization** - Strangler Fig with API Gateway facade
5. **Multi-Region API** - Regional + edge-optimized + Route 53

## Connections
- **Lambda / Step Functions / ECS**: Backend integration
- **Cognito**: User auth
- **WAF**: API protection
- **CloudWatch / X-Ray**: Monitoring & tracing
- **EventBridge**: Async fan-out

## Well-Architected
- Security: Auth, throttling, WAF, mTLS
- Performance: Caching, edge optimization
- Cost: HTTP API for low cost, REST for features
- Reliability: Canary + throttling

**SAP-C02 Focus**: Design secure, scalable, observable API layers with proper auth and throttling.