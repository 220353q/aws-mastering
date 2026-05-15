# Amazon VPC

## Overview
Amazon Virtual Private Cloud (VPC) provides isolated virtual network for AWS resources. Full control over IP addressing, subnets, routing, gateways.

## Key Features
- Subnets (Public/Private/Isolated), Route Tables, NACLs
- Internet Gateway, NAT Gateway, VPC Endpoints (Gateway/Interface)
- VPC Peering, Transit Gateway, PrivateLink
- Flow Logs, Traffic Mirroring
- IPv6, BYOIP

## Use Cases (Tier 1)
1. **Secure Multi-Tier Web App** - Public subnets (ALB), Private subnets (EC2/App), Isolated (DB)
2. **Hybrid Connectivity** - Direct Connect + VPN + Transit Gateway
3. **Microservices Isolation** - Separate VPCs per domain + Peering/TGW
4. **Serverless Private** - VPC Endpoints + Lambda in VPC
5. **Compliance Isolation** - Dedicated VPCs for regulated workloads

## Connections
- **EC2, RDS, Lambda**: Deploy in VPC
- **ELB**: Public/Private
- **Route 53**: DNS resolution
- **CloudFront**: Origin in VPC via Origin Shield/PrivateLink
- **Transit Gateway**: Hub-and-spoke multi-VPC

## Well-Architected
- Security: NACLs + Security Groups + Private subnets
- Reliability: Multi-AZ subnets
- Cost: NAT Gateway optimization, VPC Endpoints (no data transfer)
- Performance: Placement groups, enhanced networking

**SAP-C02 Focus**: Design secure, scalable network architectures with proper segmentation and connectivity.