# Pattern: Strangler Fig (Legacy Migration)

## Architecture
Facade (API Gateway / ALB) routes new features to modern services (Lambda / containers) while legacy monolith handles old paths. Gradually migrate and decommission.

## Key Services
- **API Gateway / ALB** (facade + routing)
- **Lambda / ECS / EKS** (new microservices)
- **Step Functions** (migration workflows)
- **EventBridge** (event bridging)
- **CloudWatch + X-Ray** (observability during migration)

## Benefits
- Incremental migration (no big bang)
- Continuous value delivery
- Low risk

**SAP-C02**: Recommended for monolith-to-microservices or lift-and-shift modernization.