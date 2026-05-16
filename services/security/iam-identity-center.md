# AWS IAM Identity Center

AWS IAM Identity Center は、社員や管理者が複数AWSアカウントへSSOするための中心サービス。SAP-C02では、Organizations、Permission Set、外部IdP連携、Cognitoとの違いが頻出。

## 一言で

社員がAWSアカウント/CLI/コンソールへSSOするならIAM Identity Center。アプリ利用者の認証ならCognito。

## 主要概念

| 概念 | 意味 |
|---|---|
| Permission Set | 各AWSアカウントに割り当てる権限テンプレート |
| Assignment | ユーザー/グループにアカウントとPermission Setを紐づける |
| Identity Source | IAM Identity Center内蔵、Active Directory、外部IdPなど |
| AWS Organizations | 複数アカウントへの割り当て基盤 |

## 試験で選ぶ条件

- 複数アカウントへ社員SSOを提供したい
- 外部IdPやActive Directoryと連携したい
- アカウントごとに権限セットを標準化したい
- 長期アクセスキーを減らしたい

## Cognitoとの違い

| 要件 | 選ぶ |
|---|---|
| 社員がAWSコンソール/CLIへSSO | IAM Identity Center |
| 顧客/アプリユーザーのログイン | Cognito User Pool |
| アプリユーザーへ一時AWS認証情報 | Cognito Identity Pool |

## High-Risk Exam Traps

- IAM Identity Centerのグループ名だけでSCP相当の禁止を実現できるわけではない。
- SCPは最大権限、Permission Setは付与権限。役割が違う。
- CognitoをAWS管理者SSOとして選ばない。

## Related

- [IAM](iam.md)
- [Cognito](cognito.md)
- [Control Tower](../management/controltower.md)
