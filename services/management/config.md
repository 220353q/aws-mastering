# AWS Config

## 何をするサービスか

AWS Config は、AWSリソースの設定履歴を記録し、ルールに基づいて準拠状態を評価するサービス。SAP-C02では、**設定の過去履歴、準拠評価、継続的コンプライアンス、マルチアカウント統制**で出る。

## CloudTrail / Config / CloudWatch の違い

| サービス | 見るもの | 代表的な問い |
|---|---|---|
| CloudTrail | API呼び出し | 誰が変更したか |
| AWS Config | リソース設定状態と履歴 | 何がどう変わったか、準拠しているか |
| CloudWatch | メトリクス/ログ/アラーム | 性能やアプリログがどうなっているか |

## AWS Config Rules

Config Rulesは、リソース設定がルールに準拠しているか評価する。AWS管理ルールとカスタムルールがある。

例:

- S3バケットがパブリックでないか
- EBSボリュームが暗号化されているか
- Security Groupが0.0.0.0/0でSSHを許可していないか
- RDSがMulti-AZか

## Conformance Packs

Conformance Packは、複数のConfig Rulesと修復アクションをひとまとまりにして展開する仕組み。Organizations配下に共通のコンプライアンス標準を配布する場面で重要。

```text
Security Baseline Conformance Pack
  ├─ S3 public access prohibited
  ├─ EBS encrypted
  ├─ RDS storage encrypted
  ├─ CloudTrail enabled
  └─ Remediation actions
```

## Aggregator

Config Aggregatorを使うと、複数アカウント・複数リージョンの準拠状態を集約できる。Control TowerやSecurity Hubと合わせて、大規模組織の統制に使う。

## よくある誤答

- **CloudTrailで設定準拠を継続評価する**: CloudTrailはAPI監査であり、準拠評価はConfig。
- **Configだけで攻撃を検知する**: 脅威検出はGuardDuty、所見集約はSecurity Hub。
- **Configは予防的制御だと思う**: 基本は検出的制御。予防はSCP、IAM、Service Control、CloudFormation Hookなどと組み合わせる。

## SAP-C02 Focus

- 継続的コンプライアンスにはConfig Rules。
- 組織標準をまとめて展開するならConformance Packs。
- 変更者特定はCloudTrail、設定差分はConfig、通知と自動修復はEventBridge + Systems Manager Automation / Lambda。
