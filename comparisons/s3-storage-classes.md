# Comparison: S3 Storage Classes

| Class | Retrieval Time | Durability | Cost (storage) | Best For |
|-------|----------------|------------|----------------|----------|
| **Standard** | Immediate | 11 9s | Highest | Hot data, frequent access |
| **Intelligent-Tiering** | Immediate / Auto | 11 9s | Auto-optimized | Unknown or changing access patterns |
| **Standard-IA** | Milliseconds | 11 9s | ~40% lower | Infrequent but immediate access |
| **One Zone-IA** | Milliseconds | 99.999999999% (10 9s) | Lower | Non-critical infrequent data |
| **Glacier Instant Retrieval** | Milliseconds | 11 9s | Very low | Archive with occasional access |
| **Glacier Flexible Retrieval** | 1 min - 12 hrs | 11 9s | Lowest | Long-term archive, bulk retrieval |
| **Glacier Deep Archive** | 12 hrs | 11 9s | Lowest | Compliance archive, 7-10+ years |

**Recommendation**: Start with Intelligent-Tiering. Use lifecycle policies to move cold data to Glacier/Deep Archive. Monitor with S3 Storage Lens.