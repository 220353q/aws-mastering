# AWS Savings Plans

## 何をする仕組みか

Savings Plans は、1年または3年の時間あたり利用額コミットメントと引き換えに、対象コンピュート利用の料金を下げる料金モデル。SAP-C02では、**柔軟性の高い長期割引**として問われる。

## 種類

| 種類 | 対象 | 柔軟性 | 試験での使いどころ |
|---|---|---|---|
| Compute Savings Plans | EC2, Fargate, Lambda | 高い | インスタンスファミリー/リージョン変更があり得る |
| EC2 Instance Savings Plans | EC2 | 中 | 特定リージョン・インスタンスファミリーが安定 |
| SageMaker Savings Plans | SageMaker | 対象限定 | ML利用が安定 |

## RIとの違い

| 観点 | Savings Plans | Reserved Instances |
|---|---|---|
| 基本 | $/hourの利用額コミット | 属性一致型の割引 |
| 柔軟性 | 特にCompute SPは高い | Standard RIは低め、Convertible RIは変更可 |
| 容量確保 | しない | Zonal RIは容量予約効果あり |
| 対象 | EC2/Fargate/Lambda等 | 主にEC2/RDSなどサービス別 |

## よくある誤答

- **Savings Plansを容量予約として選ぶ**: Savings Plansは料金割引であり、容量確保ではない。
- **変動が大きいワークロードに過剰コミットする**: 安定ベースラインにだけ適用する。
- **Spotと混同**: Spotは中断可能な余剰キャパシティ利用。Savings Plansは長期割引。

## SAP-C02 Focus

- まずオンデマンド実績をCost Explorerで分析。
- 安定しているベースラインにSavings Plansを適用。
- 変動部分はAuto Scaling、Spot、Serverless、オンデマンドで吸収。
