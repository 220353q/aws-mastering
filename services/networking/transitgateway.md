# AWS Transit Gateway

## Overview
Hub-and-spoke network transit hub. Connect thousands of VPCs, on-premises networks, and remote networks through a single gateway.

## Key Features
- Hub-and-spoke architecture (simplifies peering)
- Cross-account VPC attachments
- VPN / Direct Connect / Peering attachments
- Transit Gateway Route Tables + propagation
- Inter-region peering (global)
- Multicast support
- Network Manager (visualization + monitoring)

## Use Cases (Tier 1 - Very Important)
1. **Multi-VPC / Multi-Account Connectivity** - Central hub for hundreds of VPCs
2. **Hybrid Cloud** - On-prem + AWS via Direct Connect + VPN
3. **Global Network** - Inter-region Transit Gateway peering
4. **Shared Services / Inspection** - Central inspection VPC with Transit Gateway
5. **Merger & Acquisition Integration** - Quickly connect acquired company networks

## Connections
- **VPC**: Attachments
- **Direct Connect / VPN**: On-prem connectivity
- **Route 53 Resolver**: DNS across network
- **CloudWatch + Network Manager**: Monitoring
- **Security Hub / GuardDuty**: Central security

## Well-Architected
- Reliability: Multi-AZ, inter-region peering
- Security: Centralized inspection + PrivateLink
- Cost: Attachment + data processing fees (cheaper than many VPC peerings)
- Performance: High bandwidth, low latency

**SAP-C02 Focus**: Design scalable, manageable multi-account and hybrid networks with centralized control.