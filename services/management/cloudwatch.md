# Amazon CloudWatch & Related

## Overview
Amazon CloudWatch monitors resources and applications. Logs, metrics, alarms, dashboards. Paired with CloudTrail (audit), Config (compliance).

## Key Features
- Metrics, Alarms, Dashboards
- Logs Insights, Contributor Insights
- Synthetics (canaries), RUM
- CloudWatch Logs + Subscription Filters
- EventBridge integration

## Use Cases
1. **Infrastructure Monitoring** - EC2, RDS, Lambda metrics + alarms
2. **Application Observability** - Custom metrics + X-Ray traces
3. **Log Analytics** - Logs Insights for troubleshooting
4. **Compliance Auditing** - CloudTrail + Config + Security Hub
5. **Cost Anomaly Detection** - CloudWatch + Cost Explorer

## Connections
- **All Services**: Publish metrics/logs
- **EventBridge**: Trigger on alarms
- **Lambda**: Process logs
- **SNS**: Alarm notifications

**SAP-C02**: Design comprehensive observability with CloudWatch + X-Ray for performance and security monitoring.