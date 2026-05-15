# Amazon CloudFront

## Overview
Global Content Delivery Network (CDN) with edge locations. Caches content, reduces latency, provides DDoS protection via Shield.

## Key Features
- Edge Locations + Regional Edge Caches
- Origin (S3, ELB, API Gateway, EC2, Custom)
- Behaviors, Cache Policies, Origin Request/Response Policies
- Lambda@Edge / CloudFront Functions (edge compute)
- Signed URLs / Cookies, Field-Level Encryption
- Real-time metrics + logging

## Use Cases (Tier 1)
1. **Static + Dynamic Content Acceleration** - S3 + CloudFront for websites/APIs
2. **Video Streaming** - On-demand / live with low latency
3. **API Acceleration** - API Gateway + CloudFront for global APIs
4. **Security Layer** - WAF + Shield integration at edge
5. **Serverless at Edge** - Lambda@Edge for A/B testing, auth, redirects

## Connections
- **S3 / ELB / API Gateway**: Origins
- **Route 53**: DNS + failover
- **WAF / Shield**: Security at edge
- **Lambda@Edge**: Custom logic
- **CloudWatch**: Real-time monitoring

## Well-Architected
- Performance: Edge caching + compression
- Security: HTTPS, signed URLs, WAF
- Cost: Cache hit ratio optimization
- Reliability: Origin failover

**SAP-C02 Focus**: Design global, secure, high-performance content delivery with edge computing.