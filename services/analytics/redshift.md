# Amazon Redshift

## Overview
Fully managed petabyte-scale data warehouse. Columnar storage, massively parallel processing (MPP), Redshift Spectrum for S3 queries.

## Key Features
- RA3 nodes (compute/storage separation)
- Redshift Spectrum (query S3 data lake)
- Concurrency Scaling, Redshift Serverless
- Data sharing, Cross-region snapshots
- Integration with Glue Data Catalog + Lake Formation

## Use Cases (Tier 2)
1. **Enterprise Data Warehouse** - Centralized analytics + BI (QuickSight)
2. **Data Lakehouse** - Redshift + S3 + Spectrum + Glue
3. **Real-time + Batch Analytics** - Kinesis Firehose + Redshift
4. **Cost-Effective Analytics** - Serverless + Spectrum for variable workloads

## Connections
- **S3 + Glue + Athena**: Data lakehouse
- **QuickSight**: BI layer
- **Kinesis Firehose**: Streaming ingestion
- **Lake Formation**: Governance

**SAP-C02**: Design scalable analytics platforms combining warehouse + data lake.