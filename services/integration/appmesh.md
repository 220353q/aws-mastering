# AWS App Mesh

## Overview
Service mesh for containerized applications. Provides consistent visibility, traffic control, and security across services running on ECS, EKS, EC2, or Fargate.

## Key Features
- Sidecar proxy (Envoy) injection
- Traffic routing (weighted, canary, blue/green)
- Observability (metrics, logs, traces with X-Ray)
- mTLS, access control, retry policies
- Virtual services, virtual nodes, virtual routers
- Integration with CloudWatch, X-Ray, Prometheus

## Use Cases (Tier 2)
1. **Microservices Traffic Management** - Canary releases, A/B testing, circuit breaking
2. **Observability in Containers** - Consistent metrics + traces across services
3. **Security in Service Mesh** - mTLS between services without code changes
4. **Multi-Environment Routing** - Route traffic between dev/staging/prod meshes
5. **Legacy + Modern Coexistence** - Gradual migration with traffic splitting

## Connections
- **ECS / EKS / Fargate / EC2**: Workload platforms
- **X-Ray + CloudWatch**: Observability
- **App Mesh Controller**: Kubernetes integration
- **IAM**: Service identity

## Well-Architected
- Security: mTLS + access control
- Performance: Traffic shaping + retries
- Observability: Consistent tracing + metrics
- Reliability: Circuit breaking + retries

**SAP-C02 Focus**: Design observable, secure, resilient microservices with service mesh.