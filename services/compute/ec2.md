# Amazon EC2

## Overview
Amazon Elastic Compute Cloud (EC2) provides resizable compute capacity in the cloud. Launch virtual servers (instances) with various OS, CPU, memory, storage options.

## Key Features
- Instance Types (General, Compute, Memory, Storage Optimized)
- Auto Scaling
- Spot Instances, Reserved, On-Demand, Savings Plans
- Security Groups, Key Pairs, IAM Roles
- EBS Volumes, Instance Store

## Use Cases (Tier 1 - Selected 5)
1. **Web Application Hosting** - Auto Scaling + ELB + EC2 for high availability web apps (SAP frequent)
2. **Batch Processing** - Spot Instances for cost-effective large-scale data processing
3. **Development/Test Environments** - On-demand instances for CI/CD pipelines
4. **High Performance Computing (HPC)** - Cluster instances with EBS optimized
5. **Legacy Application Migration** - Lift-and-shift with minimal changes

## Connections with Other Services
- **ELB + Auto Scaling** : High availability
- **VPC** : Network isolation
- **CloudWatch** : Monitoring & alarms
- **IAM** : Secure access
- **S3** : Data storage
- **RDS** : Backend database

## Comparison
See comparisons/ec2-vs-lambda-fargate.md

## Well-Architected Alignment
- Reliability: Multi-AZ, Auto Scaling
- Security: Security Groups, IAM
- Cost: Spot, Savings Plans
- Performance: Instance right-sizing

**SAP-C02 Focus**: Design resilient, scalable compute architectures.