# AWS IAM & Organizations

## Overview
AWS Identity and Access Management (IAM) controls access to AWS services/resources. AWS Organizations for multi-account governance.

## Key Features
- Users, Groups, Roles, Policies (JSON)
- IAM Identity Center (SSO)
- Organizations: SCPs, OU, Consolidated Billing
- Access Analyzer, IAM Access Analyzer
- Permission Boundaries, Session Policies

## Use Cases (Tier 1)
1. **Least Privilege Access** - Fine-grained policies for teams/services
2. **Cross-Account Access** - AssumeRole for secure multi-account
3. **Federated Access** - SAML/OIDC with corporate IdP
4. **Service-to-Service** - IAM Roles for Lambda/EC2 to S3/DynamoDB
5. **Governance at Scale** - Organizations + SCPs for compliance (SAP common)

## Connections
- **All Services**: Every AWS resource uses IAM
- **Organizations**: Multi-account strategy
- **CloudTrail**: Audit all IAM actions
- **GuardDuty/Security Hub**: Threat detection

## Well-Architected Security Pillar
- Identity: Least privilege, no long-term credentials
- Audit: CloudTrail + Access Analyzer
- Encryption: KMS with IAM policies

**SAP-C02 Focus**: Design secure, compliant multi-account architectures with proper IAM boundaries.