# Amazon Inspector

Amazon Inspector は、EC2、コンテナイメージ、Lambdaなどの脆弱性を継続スキャンするサービス。SAP-C02では、GuardDuty/Macie/Security Hubとの違いで問われる。

## 一言で

ソフトウェア脆弱性や露出を見つけるならInspector。脅威検出はGuardDuty、機密データ検出はMacie。

## 試験で選ぶ条件

- EC2のパッケージ脆弱性を継続検出したい
- ECRイメージのCVEを検出したい
- Lambda関数の脆弱性を確認したい
- 結果をSecurity Hubへ集約したい

## 役割比較

| 要件 | サービス |
|---|---|
| 脆弱性管理 | Inspector |
| 異常なAPI/ネットワーク/DNS挙動検出 | GuardDuty |
| S3内の機密情報検出 | Macie |
| 所見集約・標準準拠チェック | Security Hub |

## High-Risk Exam Traps

- InspectorをWAFのような通信遮断サービスとして選ばない。
- GuardDutyは脆弱性スキャナではない。
- Security Hubは検出元そのものではなく、集約/評価の中心。

## Related

- [GuardDuty](guardduty.md)
- [Security Hub](../management/security-hub.md)
- [Macie](macie.md)
