# AWS Service Catalog

AWS Service Catalog は、組織が承認済みの製品/テンプレートをカタログ化し、利用者がセルフサービスで安全にプロビジョニングできるようにするサービス。

## 一言で

標準化された構成だけを、各チームにセルフサービス提供したいならService Catalog。

## 試験で選ぶ条件

- 中央ITが承認済みテンプレートを配布したい
- 開発チームに自由度を与えつつ、構成の逸脱を減らしたい
- CloudFormationテンプレートを製品として提供したい
- Control TowerのAccount Factoryやガバナンスと組み合わせたい

## CloudFormationとの違い

| 要件 | サービス |
|---|---|
| IaCテンプレートでスタックを作る | CloudFormation |
| 承認済みテンプレートをカタログとして提供 | Service Catalog |
| 複数アカウント/リージョンへ同一Stack展開 | CloudFormation StackSets |

## High-Risk Exam Traps

- Service Catalogはテンプレート実行の統制レイヤー。IaCエンジンそのものはCloudFormationなど。
- SCPのような最大権限制御ではない。
- 新規アカウントのランディングゾーン自動化はControl Towerが中心。

## Related

- [CloudFormation](cloudformation.md)
- [Control Tower](controltower.md)
- [IAM](../security/iam.md)
