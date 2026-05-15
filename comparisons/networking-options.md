# Comparison: AWS Networking Connectivity Options

| Option | Use Case | Latency | Cost | Security |
|--------|----------|---------|------|----------|
| **VPC Peering** | Simple VPC-to-VPC | Low | Low | Private |
| **Transit Gateway** | Hub-and-spoke, many VPCs/on-prem | Low-Medium | Medium | Private, scalable |
| **Direct Connect** | Hybrid, high bandwidth, consistent | Low | High (port + data) | Private, dedicated |
| **Site-to-Site VPN** | Hybrid, quick setup | Medium | Low-Medium | Encrypted tunnel |
| **PrivateLink** | Expose service privately (no public IP) | Low | Medium | Private endpoint |
| **Global Accelerator** | Global app acceleration, failover | Low (anycast) | Medium-High | Edge security |

**Recommendation**: Transit Gateway for multi-VPC/on-prem hub. Direct Connect + VPN for hybrid. PrivateLink for service exposure. Global Accelerator for global users.