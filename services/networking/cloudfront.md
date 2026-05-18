# Amazon CloudFront

## Overview
Global Content Delivery Network (CDN) with edge locations. Caches content, reduces latency, provides DDoS protection via Shield.

CloudFrontは「一方向通信サービス」ではない。HTTP/HTTPSのrequest/responseを中継し、POSTなどの動的リクエストをオリジンへ転送することもできる。ただし、CloudFrontの主目的はCDN、エッジ配信、キャッシュ、WAF連携、オリジン保護であり、IoT Coreのようなデバイス通信管理やMQTT brokerではない。

```text
Client
  → CloudFront
  → Origin
  ← CloudFront
  ← Client
```

## Key Features
- Edge Locations + Regional Edge Caches
- Origin (S3, ELB, API Gateway, EC2, Custom)
- Behaviors, Cache Policies, Origin Request/Response Policies
- Lambda@Edge / CloudFront Functions (edge compute)
- Signed URLs / Cookies, Field-Level Encryption
- Real-time metrics + logging
- WAF / Shield integration at edge
- WebSocket forwarding and dynamic request forwarding to origins

## Use Cases (Tier 1)
1. **Static + Dynamic Content Acceleration** - S3 + CloudFront for websites/APIs
2. **Video Streaming** - On-demand / live with low latency
3. **API Acceleration** - API Gateway + CloudFront for global APIs
4. **Security Layer** - WAF + Shield integration at edge
5. **Serverless at Edge** - Lambda@Edge for A/B testing, auth, redirects

## What CloudFront is not

| 誤解 | 正しい見方 |
|---|---|
| CloudFrontは一方向通信なのでPOST/APIには使えない | request/responseやPOST転送は可能 |
| CloudFrontはIoTデータ受信基盤である | IoTデバイス管理、MQTT、Device Shadow、IoT RulesはIoT Coreの領域 |
| CloudFrontはAPI Gatewayの完全な代替 | API認証、スロットリング、API Key、Usage Plan等はAPI Gatewayが得意 |
| CloudFrontはGlobal Acceleratorの代替 | CloudFrontはHTTP/HTTPSのCDN。GAはTCP/UDPのグローバル入口最適化 |

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

## SAP-C02 Focus

Design global, secure, high-performance content delivery with edge computing.

```text
Static / dynamic content acceleration
  → CloudFront

Private S3 origin protection
  → CloudFront + OAC

Global API acceleration / edge WAF
  → CloudFront + API Gateway / ALB origin

IoT device communication / MQTT / Device Shadow / IoT Rules
  → AWS IoT Core

General REST/HTTP API management
  → API Gateway
```

Protocol / AWS entrypoint の横断整理は [HTTP / HTTPS / WebSocket / MQTT / gRPC / TCP / UDP とAWS入口サービス](../../comparisons/protocols-and-aws-entrypoints.md) を参照。

## このページを読んだあとに戻るべき関連ページ

- [HTTP / HTTPS / WebSocket / MQTT / gRPC / TCP / UDP とAWS入口サービス](../../comparisons/protocols-and-aws-entrypoints.md)
- [Networking Connectivity Options](../../comparisons/networking-options.md)
- [Elastic Load Balancing](elb.md)
- [AWS Global Accelerator](global-accelerator.md)
