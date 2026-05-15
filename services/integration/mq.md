# Amazon MQ

## Overview
Managed message broker service for Apache ActiveMQ and RabbitMQ. Lift-and-shift existing messaging workloads to AWS with minimal changes.

## Key Features
- ActiveMQ 5.17+ and RabbitMQ 3.11+ (latest)
- Single-instance, active/standby, cluster deployment
- Encryption, IAM, VPC, audit logs
- MQTT, AMQP, STOMP, OpenWire, JMS support
- Cross-region replication (active/standby)

## Use Cases (Tier 2)
1. **Legacy JMS / AMQP Migration** - Move on-prem ActiveMQ/RabbitMQ to AWS
2. **Enterprise Messaging Backbone** - Reliable pub/sub and queuing
3. **Event-Driven Integration** - Bridge between systems with different protocols
4. **IoT / Industrial Messaging** - MQTT support for device communication
5. **Hybrid Messaging** - Connect on-prem and AWS messaging

## Connections
- **Lambda / ECS / EKS**: Message consumers/producers
- **EventBridge**: Event routing + transformation
- **SQS / SNS**: Modern AWS messaging (when possible)
- **CloudWatch**: Monitoring
- **VPC / Direct Connect**: Hybrid connectivity

## Well-Architected
- Reliability: Active/standby + clustering
- Security: Encryption + IAM + VPC
- Cost: Instance-based (cheaper than self-managed)
- Operational Excellence: Managed patching and updates

**SAP-C02**: Design reliable messaging for legacy modernization and hybrid integration.