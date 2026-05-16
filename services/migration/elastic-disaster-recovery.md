# AWS Elastic Disaster Recovery

AWS Elastic Disaster Recovery (AWS DRS) は、オンプレミスや他クラウドのサーバーをAWSへ継続レプリケーションし、障害時にEC2として起動するDRサービス。

## 一言で

サーバー単位のDRを、アプリ改修を抑えてAWSへ作るならAWS Elastic Disaster Recovery。移行カットオーバーが中心ならMGN。

## MGNとの違い

| 観点 | AWS Elastic Disaster Recovery | AWS Application Migration Service |
|---|---|---|
| 主目的 | DR待機環境 | 移行/リホスト |
| 使う場面 | 障害時にAWSで復旧 | 移行波、テスト、カットオーバー |
| レプリケーション | 継続レプリケーション | 継続レプリケーション |
| 試験キーワード | disaster recovery, failover, recovery drill | migration, rehost, cutover |

## 試験で選ぶ条件

- 既存サーバーを大きく変更せずDRを作りたい
- RTO/RPOを短くしたいが、常時フルサイズ稼働は避けたい
- 復旧訓練を行い、障害時にEC2へ起動したい
- オンプレや他クラウドからAWSへDRしたい

## High-Risk Exam Traps

- DRSをデータベース専用移行ツールとして選ばない。DB移行はDMS/SCTも検討する。
- Active/Activeのアプリ設計を自動的に作るサービスではない。
- 移行問題ではMGN、DR問題ではDRSという文脈を読む。

## Related

- [Application Migration Service](mgn.md)
- [Disaster Recovery Pattern](../../patterns/disaster-recovery.md)
- [Domain 4](../../sap-c02/domain4.md)
