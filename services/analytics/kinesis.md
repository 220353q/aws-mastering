# Amazon Kinesis

## Overview
Real-time streaming data platform. Ingest, process, and analyze streaming data at scale.

## Key Features
- Kinesis Data Streams (shards, records)
- Kinesis Data Firehose (delivery to S3/Redshift/Elasticsearch)
- Kinesis Data Analytics (SQL / Flink for real-time processing)
- Kinesis Video Streams

## Use Cases (Tier 2 - High Value)
1. **Real-time Log / Metric Processing** - Streams → Lambda / Kinesis Analytics → S3 / Redshift
2. **Clickstream / IoT Telemetry** - High throughput ingestion + immediate analytics
3. **Fraud Detection** - Real-time anomaly detection with Kinesis + Lambda + DynamoDB
4. **Live Dashboard** - Firehose to QuickSight / OpenSearch

## Connections
- **Lambda / Step Functions**: Processing
- **S3 / Redshift / OpenSearch**: Sinks
- **EventBridge**: Event routing
- **CloudWatch**: Monitoring

**SAP-C02**: Design real-time analytics pipelines with low latency.