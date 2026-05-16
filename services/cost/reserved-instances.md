# Reserved Instances

## 何をする仕組みか

Reserved Instances は、EC2などのオンデマンド使用量に対して、条件が一致した場合に割引を適用する料金モデル。物理的なインスタンスを購入するわけではなく、請求上の割引。

## EC2 RIの種類

| 種類 | 特徴 | 向く場面 |
|---|---|---|
| Standard RI | 割引率が高いが柔軟性は低い | 長期間変わらない安定ワークロード |
| Convertible RI | インスタンスファミリー等を変更可能 | 将来変更の可能性がある |
| Zonal RI | 特定AZに紐づき容量予約効果 | 容量確保が必要な重要ワークロード |
| Regional RI | リージョン内で柔軟に割引適用 | AZ固定不要な通常用途 |

## Capacity Reservationとの違い

| 項目 | Reserved Instance | Capacity Reservation |
|---|---|---|
| 主目的 | コスト割引 | 容量確保 |
| 期間 | 1年/3年が中心 | 任意期間 |
| 割引 | あり | 単体では割引なし |
| 組み合わせ | Capacity Reservation + Savings Plans/RI も検討 | 重要システムの容量確保 |

## SAP-C02 Focus

- 「コスト削減」ならRI/Savings Plans。
- 「特定AZに必ず起動できる容量が必要」ならZonal RIまたはCapacity Reservation。
- 「リージョンやファミリー変更の可能性」ならCompute Savings PlansやConvertible RIを検討。
