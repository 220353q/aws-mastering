# Amazon MSK

Amazon Managed Streaming for Apache Kafka (Amazon MSK) は、Apache Kafka互換のマネージドストリーミング基盤。SAP-C02では、Kinesisとの違い、既存Kafka移行、ストリーミングアーキテクチャで問われる。

## 一言で

Kafka互換が明示要件ならMSK。AWSネイティブのシンプルなストリームならKinesisを検討する。

## Kinesisとの違い

| 要件 | 選ぶ |
|---|---|
| 既存Kafkaクライアント/エコシステムを維持 | MSK |
| AWSネイティブで運用を簡素化 | Kinesis Data Streams / Firehose |
| ストリーム処理アプリをApache Flinkで実行 | Managed Service for Apache Flink |

## 試験で選ぶ条件

- オンプレKafkaをAWSへ移行したい
- Kafka API互換が必要
- トピック/パーティション/コンシューマグループの概念を維持したい
- ストリーミング基盤をマネージド化したい

## High-Risk Exam Traps

- Kafka互換が不要ならKinesisの方が簡単な選択肢になりやすい。
- MSKは分析クエリエンジンではない。処理にはFlink、Lambda、アプリなどを組み合わせる。
- EventBridgeはイベントルーティングであり、高スループットログストリーム用途とは別。

## Related

- [Kinesis](kinesis.md)
- [Flink](flink.md)
- [EventBridge](../integration/eventbridge.md)
