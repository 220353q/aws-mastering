# Amazon Managed Service for Apache Flink

Amazon Managed Service for Apache Flink は、Apache Flinkアプリケーションをマネージドに実行するストリーム処理サービス。

## 一言で

流れてくるデータに対して、状態を持つリアルタイム処理やウィンドウ集計をしたいならManaged Service for Apache Flink。

## 試験で選ぶ条件

- KinesisやMSKのストリームをリアルタイム処理したい
- ウィンドウ集計、状態管理、イベント時間処理が必要
- ストリーム処理アプリを運用負荷少なく実行したい
- Lambda単体では処理時間/状態管理が厳しい

## Kinesis / MSK / Flink

| 役割 | サービス |
|---|---|
| ストリーム取り込み | Kinesis Data Streams / MSK |
| 配信・ロード | Kinesis Data Firehose |
| 状態を持つストリーム処理 | Managed Service for Apache Flink |
| イベントルーティング | EventBridge |

## High-Risk Exam Traps

- Flinkはメッセージブローカーではない。入力元としてKinesis/MSKなどが必要。
- 単純なS3ロードならFirehoseの方が簡単。
- イベント駆動のサービス間連携はEventBridge/SNS/SQSの役割を読む。

## Related

- [Kinesis](kinesis.md)
- [MSK](msk.md)
- [EventBridge](../integration/eventbridge.md)
