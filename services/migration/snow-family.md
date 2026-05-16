# AWS Snow Family

## Overview
AWS Snow Family は、ネットワーク経由の転送が現実的でない大容量データを、物理デバイスでAWSへ移行するためのサービス群。Snowball Edge はデータ移行だけでなく、エッジ環境でのローカル処理にも使える。

---

## 使う場面

- ペタバイト級データをAWSへ移行したい。
- インターネットやDirect Connectの帯域では転送に時間がかかりすぎる。
- 遠隔地・工場・船舶・災害現場など、接続が不安定な環境でローカル処理したい。
- データをS3へ大量投入したい。

---

## 基本フロー

```
1. AWSでSnowデバイスジョブを作成
2. デバイスを受領
3. オンプレミスでデータをロード
4. デバイスをAWSへ返送
5. AWS側でS3などへインポート
6. デバイス上のデータは消去
```

---

## Snow Family vs DataSync

| 観点 | Snow Family | DataSync |
|---|---|---|
| 転送方式 | 物理搬送 | ネットワーク転送 |
| 向くデータ量 | TB〜PB級、大容量 | ネットワークで現実的に送れる範囲 |
| 接続要件 | 常時接続不要 | ネットワーク接続が必要 |
| 反復同期 | 不向き | 向いている |
| 代表キーワード | `limited bandwidth`, `petabytes`, `offline` | `NFS`, `SMB`, `scheduled transfer` |

---

## SAP-C02 頻出シナリオ

| キーワード | 正解アプローチ |
|---|---|
| `petabytes of data` | Snow Family |
| `limited network bandwidth` | Snow Family |
| `offline data transfer` | Snow Family |
| `edge processing with little connectivity` | Snowball Edge |
| `recurring online file sync` | DataSync |

---

## 誤答パターン

- **DataSyncを選ぶ**: 帯域不足でPB級を送れないならSnow。
- **Direct Connectを新設する**: 一回限りの大量移行なら、専用線よりSnowの方が要件に合う場合がある。
- **MGNを選ぶ**: サーバーを起動可能な形でリホストするならMGN。単なる大量データ搬送ならSnow。

---

## Connections

- **S3**: 主なインポート先。
- **DataSync**: Snow後の差分同期やオンライン転送で併用することがある。
- **KMS**: データ暗号化。
- **IAM**: ジョブ作成・S3アクセス管理。
- **Edge workloads**: Snowball Edge上でEC2互換インスタンスやLambda的処理を行う設計もある。
