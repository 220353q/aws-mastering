# Amazon Neptune

## Overview
Fully managed graph database service. Supports Property Graph (Gremlin) and RDF (SPARQL) models. Highly connected data queries with low latency.

## Key Features
- Property Graph + RDF support
- Gremlin (Apache TinkerPop) and SPARQL 1.1
- Up to 15 read replicas
- Global Database (cross-region)
- Encryption, IAM, VPC, audit logs
- Neptune ML (graph neural networks)
- Streams (change data capture)

## Use Cases (Tier 2 - High Value)
1. **Knowledge Graphs / Recommendation Engines** - User-item relationships, fraud rings
2. **Identity & Access Graphs** - Who knows whom, permission propagation
3. **Network / IT Asset Management** - Topology, dependency mapping
4. **Social / Collaboration Features** - Friend-of-friend, group recommendations
5. **Fraud Detection** - Pattern matching in transaction graphs

## Connections
- **Lambda / ECS**: Query and update graphs
- **S3**: Bulk load (CSV, RDF, Gremlin)
- **EventBridge**: Change events
- **CloudWatch**: Monitoring
- **IAM**: Fine-grained access

## Well-Architected
- Performance: Optimized for highly connected queries
- Reliability: Multi-AZ + Global Database
- Security: Encryption + IAM + VPC
- Cost: Pay for instances + storage + I/O

**SAP-C02**: Design graph-based solutions for relationship-heavy domains (fraud, recommendations, knowledge management).