# Well-Architected Security Pillar

## Design Principles
- Implement least privilege access
- Enable traceability (CloudTrail, X-Ray)
- Apply security at all layers (defense in depth)
- Automate security best practices (Config, GuardDuty, Security Hub)
- Protect data in transit and at rest (KMS, encryption)
- Keep people away from data (IAM, no long-term credentials)

## Key Services & Patterns
- IAM + Organizations + SCPs
- GuardDuty + Security Hub + Inspector + Macie
- KMS + Secrets Manager
- VPC + PrivateLink + WAF + Shield
- CloudTrail + Config + CloudWatch

## SAP-C02 Focus Areas
- Multi-account strategy with Organizations
- Data protection (encryption + access control)
- Incident response automation
- Compliance as code (Config rules + Security Hub)