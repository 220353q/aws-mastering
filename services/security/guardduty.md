# Amazon GuardDuty & Security Hub

## Overview
GuardDuty: Intelligent threat detection using ML, anomaly detection, integrated threat intelligence. Security Hub: Central security posture management and compliance.

## Key Features (GuardDuty)
- Continuous monitoring of CloudTrail, VPC Flow Logs, DNS logs
- ML-based anomaly detection (Crypto mining, C&C, unusual API calls)
- Findings severity + remediation guidance
- Multi-account with Organizations + delegated admin

## Key Features (Security Hub)
- Aggregates findings from GuardDuty, Inspector, Macie, WAF, etc.
- Compliance checks (CIS, PCI, NIST, etc.)
- Custom insights + automated response (EventBridge + Lambda)

## Use Cases (Tier 1)
1. **Threat Detection & Response** - GuardDuty findings → Security Hub → EventBridge → Lambda remediation
2. **Compliance Monitoring** - Security Hub + Config rules + automated remediation
3. **Multi-Account Security** - Organizations-wide visibility
4. **Insider Threat / Crypto** - GuardDuty ML detection
5. **Incident Response Playbooks** - Automated via Step Functions

## Connections
- **CloudTrail / VPC Flow Logs / DNS**: Data sources
- **EventBridge / Step Functions / Lambda**: Automated response
- **Security Hub**: Central dashboard
- **CloudWatch**: Alarms on high severity
- **IAM / Organizations**: Scope and permissions

## Well-Architected Security Pillar
- Detection: Continuous monitoring + ML
- Response: Automated playbooks
- Compliance: Built-in benchmarks

**SAP-C02 Focus**: Design proactive security posture with automated detection and response.