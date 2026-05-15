# Comparison: S3 vs EBS

| Aspect | S3 (Object) | EBS (Block) |
|--------|-------------|-------------|
| **Access** | HTTP/HTTPS, global | EC2 instance only (block device) |
| **Durability** | 99.999999999% (11 9s) | 99.999% (snapshot to S3) |
| **Scalability** | Unlimited | Up to 64 TiB per volume |
| **Performance** | High throughput, eventual consistency | Low latency, IOPS provisioned |
| **Use Case** | Static assets, data lake, backups, logs | Boot volumes, databases, high IOPS apps |
| **Cost** | Per GB + requests | Per GB provisioned + IOPS |

**Recommendation**: S3 for durable object storage, EBS for low-latency block attached to EC2.