# Amazon DynamoDB

## Overview
Fully managed NoSQL database with single-digit millisecond latency at any scale. Serverless, automatic scaling, global tables for multi-region.

## Key Features
- Key-Value + Document model
- Global Tables (multi-master replication)
- On-demand / Provisioned capacity
- DynamoDB Streams, Global Secondary Indexes (GSI), Local Secondary Indexes (LSI)
- DAX (in-memory cache), TTL, Point-in-Time Recovery
- Encryption, IAM fine-grained access, VPC Endpoints

## Use Cases (Tier 1 - Selected 5)
1. **User Profiles / Session Store** - High throughput, low latency key-value access (most common)
2. **IoT / Time-series Data** - High write throughput with TTL for expiration
3. **Real-time Leaderboards / Gaming** - Atomic counters, conditional writes
4. **E-commerce Cart / Inventory** - High concurrency updates with transactions
5. **Serverless Backend** - Lambda + DynamoDB for event-driven apps (SAP core pattern)

## Connections
- **Lambda + API Gateway**: Primary serverless backend
- **EventBridge / Kinesis**: Event sourcing + streams
- **DAX**: Caching layer
- **S3**: Large objects / cold data
- **CloudWatch**: Metrics + Contributor Insights

## Comparison
See comparisons/dynamodb-vs-aurora.md

## Well-Architected
- Scalability: Automatic, global tables
- Cost: On-demand for spiky, reserved for steady
- Security: Encryption + fine-grained IAM + VPC endpoints
- Reliability: Global tables + PITR + Streams

**SAP-C02 Focus**: Design highly scalable, low-latency NoSQL backends with global distribution.