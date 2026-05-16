# Amazon Macie

Amazon Macie は、S3内の機密データを検出し、データセキュリティリスクを可視化するサービス。SAP-C02では、PII/個人情報/機密データ検出のキーワードで選ぶ。

## 一言で

S3の中身に機密情報が含まれるか見つけたいならMacie。

## 試験で選ぶ条件

- S3バケット内のPIIや機密データを検出したい
- 組織内アカウントのS3データリスクを集約したい
- 検出結果をSecurity HubやEventBridgeへ連携したい
- 公開設定だけでなく、データ内容に基づくリスクも見たい

## 役割比較

| 要件 | サービス |
|---|---|
| S3の機密データ検出 | Macie |
| S3公開設定や暗号化準拠 | Config / Security Hub |
| API操作監査 | CloudTrail |
| 脅威検出 | GuardDuty |

## High-Risk Exam Traps

- MacieはS3中心。任意DBの列レベル権限管理ならLake Formationなどを考える。
- 暗号化は機密データの存在検出を代替しない。
- Security HubはMacie所見を集約できるが、Macieの分類機能そのものではない。

## Related

- [S3](../storage/s3.md)
- [Security Hub](../management/security-hub.md)
- [Lake Formation](../analytics/lakeformation.md)
