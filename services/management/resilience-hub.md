# AWS Resilience Hub

AWS Resilience Hub は、アプリケーションの回復性を評価し、RTO/RPO目標に対するリスクや改善案を確認するサービス。

## 一言で

アプリがRTO/RPOを満たせるか評価し、改善計画を作るならResilience Hub。

## 試験で選ぶ条件

- 既存アプリケーションの回復性を継続評価したい
- RTO/RPO目標に対して構成が十分か確認したい
- DRテストや障害注入と組み合わせたい
- Well-Architected Reliabilityの改善を運用に落としたい

## 関連サービスとの違い

| 要件 | サービス |
|---|---|
| 回復性評価と改善推奨 | Resilience Hub |
| 障害注入実験 | Fault Injection Simulator |
| バックアップ統制 | AWS Backup |
| サーバーDR待機 | Elastic Disaster Recovery |

## High-Risk Exam Traps

- Resilience HubはDR先環境そのものを自動構築するサービスではない。
- FISは実験実行、Resilience Hubは評価と推奨の中心。
- RTO/RPOはビジネス要件として明示し、構成と手順で満たす。

## Related

- [FIS](fis.md)
- [Disaster Recovery](../../patterns/disaster-recovery.md)
- [Elastic Disaster Recovery](../migration/elastic-disaster-recovery.md)
