# AWS Fault Injection Simulator (FIS)

## Overview
Fully managed chaos engineering service. Run fault injection experiments to improve resilience and validate recovery procedures.

## Key Features
- Pre-built fault templates (EC2, RDS, EKS, Lambda, etc.)
- Custom fault injection (latency, packet loss, CPU stress, etc.)
- Experiment templates with stop conditions
- Integration with CloudWatch alarms (automatic stop)
- Multi-account experiments
- Experiment reports and history

## Use Cases (Tier 2 - High Value for Reliability)
1. **Chaos Engineering for Critical Workloads** - Validate Multi-AZ, failover, DR
2. **GameDay / Resilience Testing** - Scheduled chaos experiments
3. **Well-Architected Reliability Reviews** - Prove recovery procedures work
4. **Incident Response Training** - Simulate failures and practice response
5. **SLA Validation** - Measure actual RTO/RPO under failure

## Connections
- **CloudWatch**: Alarms as stop conditions + metrics
- **EventBridge**: Experiment triggers
- **Step Functions**: Orchestrate complex experiments
- **X-Ray**: Distributed tracing during experiments
- **Systems Manager**: Automation during experiments

## Well-Architected Reliability Pillar
- Test recovery procedures (core principle)
- Anticipate failure
- Learn from operational failures

**SAP-C02 Focus**: Design and validate resilient architectures with chaos engineering.