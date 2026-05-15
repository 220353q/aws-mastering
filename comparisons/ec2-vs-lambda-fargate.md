# Comparison: EC2 vs Lambda vs Fargate

| Aspect | EC2 | Lambda | Fargate |
|--------|-----|--------|---------|
| **Management** | Full control (OS, patches) | None (serverless) | None (containers) |
| **Scaling** | Manual / Auto Scaling | Automatic, instant | Automatic, per task |
| **Cost Model** | Per hour/second (reserved/spot) | Per request + duration (ms) | Per vCPU/memory per second |
| **Use Case** | Long-running, stateful, custom OS | Short bursts, event-driven | Containerized apps, microservices |
| **SAP Example** | Legacy ERP lift-and-shift | API backend, data processing | Containerized web apps, batch jobs |
| **Best For** | Fine-grained control, HPC | Cost optimization, scalability | Modern container workloads |

**Recommendation**: Use Lambda for new serverless, Fargate for containers, EC2 for control needed apps.