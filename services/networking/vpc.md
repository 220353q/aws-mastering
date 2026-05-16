# Amazon VPC

## Overview
Amazon Virtual Private Cloud (VPC) provides isolated virtual network for AWS resources. Full control over IP addressing, subnets, routing, gateways.

## Key Features
- Subnets (Public/Private/Isolated), Route Tables, NACLs
- Internet Gateway, NAT Gateway, VPC Endpoints (Gateway/Interface)
- VPC Peering, Transit Gateway, PrivateLink
- Flow Logs, Traffic Mirroring
- IPv6, BYOIP

## Security Boundary Basics

| レイヤー | 代表 | 試験での見方 |
|---|---|---|
| Resource / ENI | Security Group | Stateful、Allowのみ、戻り通信は自動許可 |
| Subnet | Network ACL | Stateless、Allow/Deny、番号順、戻り通信も明示許可 |
| Route | Route Table | 宛先に対する次ホップ。Firewallではない |
| VPC boundary | Network Firewall / GWLB | 中央検査、stateful/stateless検査、アプライアンス挿入 |
| Web edge | AWS WAF | HTTP/HTTPSのL7保護 |

```text
Internet
  │
IGW
  │
Route Table decides next hop
  │
Subnet NACL checks inbound/outbound
  │
ENI Security Group checks allowed traffic
  │
EC2 / ALB / RDS
```

詳しくは [Security Group / NACL / Firewall Decision Guide](../../comparisons/network-security-boundaries.md) と [Networking Foundations Deep Dive](../../comparisons/networking-foundations-deep-dive.md) を参照。

## Gateway Family

| Gateway | 一言 |
|---|---|
| Internet Gateway | VPCとインターネットの出入口 |
| NAT Gateway | Private subnetから外向き通信を出す |
| Transit Gateway | 多数VPC/オンプレのハブ |
| Virtual Private Gateway | VPC側のVPN/DX終端 |
| Customer Gateway | オンプレ側VPN装置の定義 |
| Direct Connect Gateway | DXを複数VPC/TGWへ広げる中継 |
| Gateway Endpoint | S3/DynamoDBへのprivate近道 |
| Gateway Load Balancer | Firewall/IDS/IPSアプライアンス挿入 |

詳しくは [AWS Gateway Services and Terms](../../comparisons/aws-gateways.md) と [Networking Foundations Deep Dive](../../comparisons/networking-foundations-deep-dive.md) を参照。

## Use Cases (Tier 1)
1. **Secure Multi-Tier Web App** - Public subnets (ALB), Private subnets (EC2/App), Isolated (DB)
2. **Hybrid Connectivity** - Direct Connect + VPN + Transit Gateway
3. **Microservices Isolation** - Separate VPCs per domain + Peering/TGW
4. **Serverless Private** - VPC Endpoints + Lambda in VPC
5. **Compliance Isolation** - Dedicated VPCs for regulated workloads

## Connections
- **EC2, RDS, Lambda**: Deploy in VPC
- **ELB**: Public/Private
- **Route 53**: DNS resolution
- **CloudFront**: Origin in VPC via Origin Shield/PrivateLink
- **Transit Gateway**: Hub-and-spoke multi-VPC

## Well-Architected
- Security: NACLs + Security Groups + Private subnets
- Reliability: Multi-AZ subnets
- Cost: NAT Gateway optimization, VPC Endpoints (no data transfer)
- Performance: Placement groups, enhanced networking

## SAP-C02での読み方

VPC問題は、まず「経路」と「許可」を分ける。Route Tableは道案内、Security Groupはリソース単位のAllow、NACLはサブネット単位のAllow/Deny。S3/KMS/AWS APIが絡む場合は、ネットワーク到達性だけでなくIAM、resource policy、KMS key policyも見る。

```text
通信できない?
  → route / gateway / endpoint はある?
  → NACL は戻り通信を許可している?
  → SG は送信元/宛先を許可している?
  → AWS APIならIAM/resource policy/KMSは許可している?
```

## このページを読んだあとに戻るべき関連ページ

- [Networking Foundations Deep Dive](../../comparisons/networking-foundations-deep-dive.md)
- [Security Group / NACL / Firewall Decision Guide](../../comparisons/network-security-boundaries.md)
- [AWS Gateway Services and Terms](../../comparisons/aws-gateways.md)
- [Networking Connectivity Options](../../comparisons/networking-options.md)
- [PrivateLink](privatelink.md)
- [Transit Gateway](transitgateway.md)
