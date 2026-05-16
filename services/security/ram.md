# AWS Resource Access Manager

AWS Resource Access Manager (AWS RAM) は、AWSリソースを複数アカウントやOrganizations内で共有するサービス。

## 一言で

サブネット、Transit Gateway、Route 53 Resolver Rule などのリソースをアカウント間で共有したいならRAM。

## 試験で選ぶ条件

- 複数アカウントで共有VPCサブネットを使いたい
- Transit Gatewayを他アカウントへ共有したい
- Route 53 Resolver RuleやLicense Manager設定を共有したい
- Organizations内で中央管理された共有を行いたい

## よく出る共有例

| リソース | 試験での文脈 |
|---|---|
| VPC subnet | Shared VPC、ネットワークアカウント集中管理 |
| Transit Gateway | ハブ&スポーク接続を複数アカウントへ展開 |
| Route 53 Resolver Rule | ハイブリッドDNSルール共有 |
| License configuration | ライセンス統制 |

## High-Risk Exam Traps

- RAMは権限認可全般ではない。IAM/SCP/Resource policyと組み合わせる。
- リソース共有とネットワーク到達性は別。ルートやSG/NACLも確認する。
- クロスアカウントアクセスでKMS暗号化が絡む場合、KMS key policyも必要。

## Related

- [Transit Gateway](../networking/transitgateway.md)
- [VPC](../networking/vpc.md)
- [IAM](iam.md)
