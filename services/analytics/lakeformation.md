# AWS Lake Formation

## Overview
Fully managed service to build, secure, and manage data lakes. Simplifies data lake creation with governance, access control, and ETL.

## Key Features
- Data lake creation in minutes (S3 + Glue Catalog)
- Fine-grained access control (row/column level)
- Data filtering, masking, row-level security
- Integration with Glue, Athena, Redshift, QuickSight, SageMaker
- Blueprint for common data lake patterns
- Cross-account sharing

## Use Cases (Tier 2)
1. **Enterprise Data Lake Governance** - Centralized access control across teams
2. **Self-Service Analytics** - Data lake with row/column security
3. **Data Mesh / Domain-Oriented** - Cross-account sharing with Lake Formation
4. **Compliance & Audit** - Fine-grained access + CloudTrail integration
5. **ML Feature Store** - Lake Formation + SageMaker Feature Store

## Connections
- **S3 + Glue + Athena + Redshift**: Core data lake stack
- **IAM + Organizations**: Access control
- **CloudTrail + Security Hub**: Audit & compliance
- **SageMaker**: Feature store integration

## Well-Architected
- Security: Fine-grained access control + encryption
- Governance: Centralized policy + audit
- Cost: S3 + Athena pay-per-query
- Reliability: S3 durability + Glue ETL

**SAP-C02**: Design governed, secure, self-service data lakes with fine-grained access control.