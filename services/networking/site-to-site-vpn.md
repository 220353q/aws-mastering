# AWS Site-to-Site VPN

## Positioning

AWS Site-to-Site VPN は、オンプレミスネットワークとAWSを **IPsec VPNトンネル** で接続するサービス。SAP-C02では、Direct Connectとの使い分け、バックアップ回線、暗号化要件、Transit Gateway連携が頻出。

---

## Core Concepts

| 概念 | 内容 |
|---|---|
| Customer Gateway | オンプレ側VPNデバイスまたはその定義 |
| Virtual Private Gateway | VPC側のVPN終端 |
| Transit Gateway | 複数VPC/オンプレを集約するVPN終端にもなる |
| VPN Tunnel | 各VPN接続は冗長な2本のトンネルを持つ |
| BGP | 動的ルーティング、冗長経路制御に有効 |
| Static Routes | 小規模・固定経路で使う場合あり |

---

## Typical Patterns

### 1. Simple VPC VPN

```text
On-premises DC
  → Customer Gateway
  → IPsec VPN
  → Virtual Private Gateway
  → VPC
```

少数VPCとの接続なら単純。ただしVPC数が増えると管理が複雑になる。

### 2. Transit Gateway VPN

```text
On-premises DC
  → IPsec VPN
  → Transit Gateway
    → VPC A
    → VPC B
    → VPC C
```

複数VPC・複数アカウントへ接続するならTGWが自然。

### 3. Direct Connect + VPN

```text
On-premises DC
  → Direct Connect
  → Private dedicated path
  → Site-to-Site VPN over DX
  → Transit Gateway / VPC
```

専用線の安定性とIPsec暗号化を同時に満たしたい場合。

---

## VPN vs Direct Connect

| 要件 | 選択 |
|---|---|
| すぐ接続したい、低コスト | Site-to-Site VPN |
| 暗号化トンネルが必要 | Site-to-Site VPN |
| 安定帯域・低ジッター・専用接続 | Direct Connect |
| 専用線 + end-to-end IPsec暗号化 | Direct Connect + Site-to-Site VPN |
| 多数VPCへ集約接続 | Transit Gateway + VPN/DX |

---

## SAP-C02 Focus

Site-to-Site VPNを選ぶ文脈:

- quick hybrid connectivity
- encrypted connection over the internet
- backup connection for Direct Connect
- small or temporary hybrid connectivity
- TGW VPN attachment for many VPCs

Direct Connectを選ぶ文脈:

- consistent bandwidth
- predictable latency
- high throughput
- private dedicated connection

---

## Exam Traps

- Direct Connect自体はIPsec暗号化トンネルではない。暗号化が明示されたらVPN over DXなどを検討。
- VPNはインターネット経由なので、専用帯域・安定遅延が最重要ならDX。
- 単一VPN接続には2本のトンネルがあるため、両方設定して冗長性を確保する。
- 複数VPCへの接続は、VPCごとにVPNを張るよりTransit Gateway集約が自然。

---

## Related

- [AWS Direct Connect](direct-connect.md)
- [AWS Transit Gateway](transitgateway.md)
- [Amazon VPC](vpc.md)
- [Networking Options Comparison](../../comparisons/networking-options.md)

## Official Docs

- https://docs.aws.amazon.com/vpn/latest/s2svpn/how_it_works.html
- https://docs.aws.amazon.com/vpn/latest/s2svpn/VPNTunnels.html
