# SAP-C02 Weakness Diagnosis

模試やシナリオ問題を解いた後、間違えた理由をサービス名ではなく「判断軸」で記録する。SAP-C02は暗記量より、似たサービスを切る力で差がつく。

## Domain Score Sheet

| Domain | 問題数 | 正解数 | 正答率 | 復習先 |
|---|---:|---:|---:|---|
| Domain 1: Organizational Complexity |  |  |  | `sap-c02/domain1.md`, `services/security/`, `services/management/` |
| Domain 2: New Solutions |  |  |  | `sap-c02/domain2.md`, `patterns/`, `comparisons/` |
| Domain 3: Continuous Improvement |  |  |  | `sap-c02/domain3.md`, `services/cost/`, `services/management/` |
| Domain 4: Migration and Modernization |  |  |  | `sap-c02/domain4.md`, `services/migration/` |

## Mistake Type Checklist

| 誤答タイプ | 典型例 | 戻る場所 |
|---|---|---|
| サービスの責務混同 | Security Hubを脅威検出そのものとして選ぶ | `comparisons/management-governance.md` |
| レイヤー混同 | WAFでTCP/UDPを保護しようとする | `comparisons/edge-security.md`, `glossary/network-web.md` |
| 権限モデル不足 | S3/IAMだけ見てKMS key policyを忘れる | `services/security/kms.md` |
| ネットワーク到達性の読み違い | PrivateLinkをVPC間フル接続として扱う | `comparisons/networking-options.md` |
| DR要件の読み違い | RTO/RPOが短いのにBackup & Restoreを選ぶ | `patterns/disaster-recovery.md` |
| 移行対象の読み違い | DB移行にMGN、ファイル移行にDMSを選ぶ | `services/migration/README.md` |
| コスト最適化の順序違い | 先にRI/SPを買い、Rightsizingを後回しにする | `comparisons/cost-optimization.md` |
| データ分析役割混同 | QuickSightだけでETL/権限制御/カタログを代替する | `comparisons/analytics-data-lake.md` |
| 認証と認可の混同 | MFA済みなら列権限も自動で満たすと考える | `glossary/network-web.md`, `services/security/cognito.md` |
| 運用サービス混同 | CloudTrailで設定準拠を評価しようとする | `services/management/README.md` |

## Review Template

```text
問題:
選んだ答え:
正解:
落とした制約語:
誤答タイプ:
なぜそのサービスではないか:
次に戻るページ:
一文で言い直す:
```

## Passing Buffer Rule

- 本番前に `full-mock-*` と `scenario-set-*` の合算で85%以上を目指す。
- 各Domainで80%未満がある場合、そのDomainの比較表と構成図へ戻る。
- 複数正解問題で1つ漏らす場合は、サービス単体ではなく「組み合わせパターン」を復習する。
