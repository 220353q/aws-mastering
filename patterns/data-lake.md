# Pattern: Serverless Data Lake

## Architecture
S3 (Raw Zone) → Glue Crawler + Athena (Query) → Redshift Spectrum / QuickSight (Analytics)

## Key Services
- **Amazon S3**: Central storage (raw, curated, processed zones)
- **AWS Glue**: ETL, Data Catalog
- **Amazon Athena**: Serverless SQL queries on S3
- **Amazon Redshift**: Data warehouse (Spectrum for S3)
- **Amazon QuickSight**: BI dashboards
- **AWS Lake Formation**: Governance, security

## Use Cases
- Log analytics, IoT data, clickstream, financial reporting
- SAP: Real-time analytics on operational data

## Best Practices
- Partition data by date/region
- Use Parquet/ORC format
- Glue ETL for transformation
- Lake Formation for access control

**Cost Optimization**: S3 Intelligent-Tiering + Athena pay-per-query.