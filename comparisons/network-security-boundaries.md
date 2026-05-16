# Security Group / NACL / Firewall Decision Guide

SAP-C02のネットワーク問題では、`Security Group`、`Network ACL`、`Route Table`、`AWS Network Firewall`、`AWS WAF` を混同しないことが重要。特に **インバウンド/アウトバウンド、Stateful/Stateless、Allow/Deny** の違いを押さえる。

## まず一言で

| 仕組み | 覚え方 | 主な役割 |
|---|---|---|
| Security Group | インスタンスの門番 | ENI/リソース単位の許可制御 |
| Network ACL | サブネットの改札 | サブネット境界のAllow/Deny制御 |
| Route Table | 道案内 | 宛先CIDRごとの次ホップ決定 |
| AWS Network Firewall | 検問所 | VPC境界/中央検査VPCでの詳細検査 |
| AWS WAF | Web受付の警備 | HTTP/HTTPSのL7攻撃対策 |

## Inbound / Outbound

視点は「そのリソースまたはサブネットに対して入るか出るか」。

```text
Client 10.0.1.10  ── request:443 ──>  Server 10.0.2.20

Client側:
  Outbound = 10.0.2.20:443 へ出る通信
  Inbound  = Serverから戻る通信

Server側:
  Inbound  = Clientから443へ入る通信
  Outbound = Clientへ戻る通信
```

## Security Group

Security GroupはENIに紐づくstatefulな仮想ファイアウォール。

| 項目 | 内容 |
|---|---|
| 適用単位 | ENI / EC2 / ALB / RDS / Lambda ENIなど |
| 状態 | Stateful |
| ルール | Allowのみ |
| Deny | 明示Denyはできない |
| 戻り通信 | 自動的に許可される |
| 評価順 | 順番なし。許可ルールの集合として評価 |

### SGの典型例

```text
ALB Security Group
  Inbound:
    443 from 0.0.0.0/0
  Outbound:
    8080 to App SG

App Security Group
  Inbound:
    8080 from ALB SG
  Outbound:
    5432 to DB SG

DB Security Group
  Inbound:
    5432 from App SG
```

ポイント:
- SGはCIDRだけでなく、別SGを送信元/宛先に指定できる。
- SG参照は「そのSGが付いたリソース群から」を意味する。
- 戻り通信のためにephemeral portを手動で開ける必要はない。

## Network ACL

Network ACLはサブネットに紐づくstatelessな境界制御。

| 項目 | 内容 |
|---|---|
| 適用単位 | Subnet |
| 状態 | Stateless |
| ルール | Allow と Deny |
| Deny | 明示Deny可能 |
| 戻り通信 | 明示的に許可が必要 |
| 評価順 | ルール番号の小さい順に最初に一致したもの |

### NACLの典型例

```text
Public Subnet NACL
  Inbound:
    100 ALLOW TCP 443 from 0.0.0.0/0
    110 ALLOW TCP 1024-65535 from 0.0.0.0/0
    *   DENY  ALL

  Outbound:
    100 ALLOW TCP 443 to 0.0.0.0/0
    110 ALLOW TCP 1024-65535 to 0.0.0.0/0
    *   DENY  ALL
```

ポイント:
- NACLはstatelessなので戻り通信のephemeral portも考える。
- 明示Denyで特定IPをブロックしたいときに使える。
- ルール番号の若いものが先。`100 DENY` が先に一致したら後続Allowは効かない。

## Default Behavior

| 対象 | 初期状態のイメージ |
|---|---|
| 新規Security Group | Inboundなし、Outbound全許可 |
| Default Security Group | 同じSGからのInboundを許可、Outbound全許可 |
| Default Network ACL | Inbound/Outbound全許可 |
| Custom Network ACL | Inbound/Outbound全拒否から開始 |

## Denyをどこで使うか

| やりたいこと | 選ぶ |
|---|---|
| リソース単位で必要な通信だけ許可 | Security Group |
| サブネット単位で特定IP/CIDRを明示拒否 | Network ACL |
| 多数VPCの通信を中央で詳細検査 | Network Firewall / GWLB |
| HTTPのSQLi/XSS/Bot/Rate制限 | AWS WAF |
| アカウント全体でAPI操作を禁止 | SCP |
| IAM主体の権限を明示Deny | IAM policy |

## Route TableはFirewallではない

Route Tableは「どこへ送るか」を決めるだけで、「許可/拒否」のセキュリティ制御ではない。

```text
Route Table:
  0.0.0.0/0 → Internet Gateway

これは「外へ行ける道がある」という意味。
実際に通信できるかは SG / NACL / Firewall / OS firewall / 相手側設定も見る。
```

## Ephemeral Port

Ephemeral portは、クライアント側が戻り通信を受けるために一時的に使う高番ポート。

```text
Client:49152  ──>  Server:443
Client:49152  <──  Server:443
```

SGはstatefulなので戻り通信を自動許可する。NACLはstatelessなので、戻り通信のポート範囲も明示的に許可する必要がある。

## Exam Traps

| 罠 | 正しい判断 |
|---|---|
| SGで明示Denyする | SGはAllowのみ。DenyはNACL/IAM/SCP/WAF/Network Firewallなど |
| NACLで戻り通信を忘れる | NACLはstateless。Inbound/Outbound両方を見る |
| Route Tableで通信を拒否する | Route Tableは道案内。拒否制御ではない |
| WAFでTCP/UDP全般を防御する | WAFはHTTP/HTTPS L7。TCP/UDPはNACL/SG/Network Firewall/Shieldなど |
| NACLだけでアプリ単位制御する | NACLはSubnet単位。リソース単位はSG |
| SG参照をCIDRのように誤解する | SG参照は「そのSGが付いたENIから/へ」 |

## 暗記テクニック

```text
SG = Stateful Gate
  S が2つ: Security Group / Stateful
  戻り通信は覚えている
  ルールはAllowだけ

NACL = Numbered ACL
  N は Number のN
  番号順に評価
  No memory = Stateless
  Denyあり

Route Table = Road Table
  道を選ぶだけ。門番ではない。

WAF = Web Application Firewall
  WebのW。HTTPの世界。
```

語呂:
- **SGは「許す門番」**: Allowだけ。帰り道は顔パス。
- **NACLは「番号つき改札」**: 番号順、拒否あり、帰りも切符が必要。
- **Routeは「地図」**: 地図は通行許可証ではない。

設問を解く順番:
1. どの単位で守りたいか: ENI、Subnet、VPC境界、Web、AWS API。
2. Denyが必要か: 必要ならSG単体ではない。
3. 戻り通信を自分で開ける必要があるか: NACLなら必要。
4. 通信の中身を見る必要があるか: HTTPならWAF、L3/L4/L7ネットワーク検査ならNetwork Firewall。

## Quick Decision Flow

```text
リソース単位で通信を絞る?
  → Security Group

サブネット単位で明示Denyしたい?
  → Network ACL

VPC間/インターネット向け通信を中央検査したい?
  → Network Firewall / Gateway Load Balancer

HTTPリクエスト内容を見てSQLi/Bot/Rate制限?
  → AWS WAF

API操作そのものを禁止?
  → SCP / IAM explicit Deny
```

## Related

- [Networking Foundations Deep Dive](networking-foundations-deep-dive.md)
- [Amazon VPC](../services/networking/vpc.md)
- [Networking Options](networking-options.md)
- [Edge Security](edge-security.md)
- [Network Firewall](../services/security/network-firewall.md)

## SAP-C02での読み方

SG/NACL問題は、守る単位と戻り通信で切る。ENI/リソース単位ならSG、サブネット単位でDenyや番号順評価が必要ならNACL、通信内容を中央検査するならNetwork Firewall/GWLB、HTTPの攻撃やrate制限ならWAF。

## このページを読んだあとに戻るべき関連ページ

- [Networking Foundations Deep Dive](networking-foundations-deep-dive.md)
- [AWS Gateway Services and Terms](aws-gateways.md)
- [Amazon VPC](../services/networking/vpc.md)
