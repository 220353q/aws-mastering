# AWS Systems Manager (SSM)

## Overview
Unified operations hub for AWS and on-premises resources. Patch management, automation, inventory, session management, parameter store, and more.

## Key Features
- Session Manager (secure shell without SSH keys)
- Patch Manager (OS patching across fleet)
- Automation (runbooks for common tasks)
- Parameter Store (secure secrets + config)
- Inventory (software/hardware inventory)
- State Manager (desired state configuration)
- OpsCenter (operational issues hub)
- Explorer (resource grouping + compliance)

## Use Cases (Tier 1 - Very Important)
1. **Secure Remote Access** - Session Manager (no bastion hosts, no SSH keys)
2. **Automated Patching** - Patch Manager across EC2 + on-prem
3. **Configuration Management** - State Manager + Automation documents
4. **Secrets & Config Management** - Parameter Store + Secrets Manager integration
5. **Operational Excellence** - Runbooks + OpsCenter for incident response

## Connections
- **EC2 / On-Premises**: Managed nodes
- **IAM**: Instance profiles + permissions
- **CloudWatch + EventBridge**: Monitoring + automation triggers
- **Secrets Manager**: Parameter Store integration
- **GuardDuty / Security Hub**: Security findings integration

## Well-Architected Operational Excellence Pillar
- Automate operations (runbooks, patching)
- Manage change (desired state)
- Learn from failures (OpsCenter)

**SAP-C02 Focus**: Design automated operations, secure access, and configuration management at scale.