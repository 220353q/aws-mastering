# Amazon OpenSearch Service

## Overview
Fully managed Elasticsearch and OpenSearch. Search, log analytics, application search, vector search, security analytics.

## Key Features
- OpenSearch (Elasticsearch fork) + Kibana / OpenSearch Dashboards
- Vector search (for RAG / semantic search)
- Security analytics, log analytics, application search
- UltraWarm + Cold storage tiers
- Cross-cluster search, replication
- Fine-grained access control, encryption, VPC
- Serverless option (new)

## Use Cases (Tier 2 - High Value)
1. **Log Analytics & Observability** - Centralized logs + dashboards (very common)
2. **Application Search** - Product catalog, knowledge base, site search
3. **Security Analytics / SIEM** - Threat detection with security analytics
4. **Vector Search for GenAI** - RAG with Bedrock + OpenSearch vector index
5. **Real-time Analytics** - UltraWarm for cost-effective retention

## Connections
- **Kinesis Firehose / Lambda**: Data ingestion
- **CloudWatch Logs**: Subscription filter to OpenSearch
- **S3**: Cold storage / bulk load
- **Bedrock / SageMaker**: Vector embeddings for RAG
- **GuardDuty / Security Hub**: Security analytics
- **IAM + Fine-grained access**: Security

## Well-Architected
- Performance: Vector search + UltraWarm
- Cost: UltraWarm + Cold tiers + serverless
- Security: Fine-grained access + encryption + VPC
- Reliability: Multi-AZ + cross-cluster replication

**SAP-C02**: Design search, log analytics, and vector search solutions for observability and GenAI.