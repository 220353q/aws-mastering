# Amazon Route 53

## Overview
Highly available and scalable DNS web service. Domain registration, DNS routing, health checks, and traffic management.

## Key Features
- DNS Routing Policies (Simple, Weighted, Latency, Failover, Geolocation, Geoproximity, IP-based, Multivalue)
- Health Checks + Failover
- Traffic Flow + Geoproximity
- Domain Registration + DNSSEC
- Resolver (inbound/outbound for hybrid)
- Global Accelerator integration

## Use Cases (Tier 1)
1. **Global Low-Latency Routing** - Latency-based or Geoproximity for worldwide apps
2. **Disaster Recovery** - Failover routing between primary/DR regions
3. **Blue/Green Deployments** - Weighted routing for gradual traffic shift
4. **Hybrid DNS** - Resolver for on-prem + AWS name resolution
5. **Multi-Region Active-Active** - Geolocation + latency policies

## Connections
- **CloudFront / ELB / Global Accelerator**: Origin / load balancer targets
- **VPC**: Private hosted zones + Resolver
- **Health Checks**: Integrate with CloudWatch alarms
- **S3 / API Gateway**: Static / API endpoints

## Well-Architected
- Reliability: Health checks + failover
- Performance: Latency / geoproximity routing
- Cost: Pay for queries + health checks
- Security: DNSSEC + private zones

**SAP-C02 Focus**: Design global, resilient DNS and traffic management strategies.