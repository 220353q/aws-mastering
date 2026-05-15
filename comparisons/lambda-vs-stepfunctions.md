# Comparison: Lambda vs Step Functions

| Aspect | Lambda | Step Functions |
|--------|--------|----------------|
| **Scope** | Single function | Multi-step workflow orchestration |
| **State** | Stateless (external store needed) | Built-in state machine (persistent) |
| **Error Handling** | Try-catch in code | Native retries, catch, choice states |
| **Duration** | Max 15 min (standard) | Up to 1 year (standard workflows) |
| **Cost** | Per request + ms | Per state transition + duration |
| **Use Case** | Single task, event handler | Complex workflows, human approval, Saga |
| **SAP Example** | API endpoint, S3 trigger | Order processing, multi-service transaction |

**Recommendation**: Lambda for simple, short tasks. Step Functions for orchestration, state, long-running or multi-step processes.