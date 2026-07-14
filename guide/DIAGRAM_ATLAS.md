# 図解アトラス — AWS構成を人間が読める形で描く

図はサービスアイコンを並べるためではない。

文章だけでは保持しにくい、**流れ・境界・状態・時間・障害**を外に出すために使う。

---

# 1. 図を描く前に決める

一枚の図で答える問いを一つにする。

悪い例：

> AWS全体構成

良い例：

- User requestはどこを通るか
- Order eventは誰へ配送されるか
- Dataの正本とReplicaはどこか
- AZ障害時に何が切り替わるか
- Human userはどうRoleを取得するか
- Migration cutoverでTrafficとDataがどう動くか

---

# 2. 矢印の文法

矢印には動詞または運ばれるものを書く。

| Label | 意味 |
|---|---|
| `HTTPS request` | 同期Request |
| `DNS query / answer` | 名前解決 |
| `message` | Queueへ預けるWork |
| `event` | 発生した事実の通知 |
| `invoke` | Function / Task起動 |
| `SQL read/write` | DB操作 |
| `replication` | 現在状態の複製 |
| `backup copy` | 復旧用履歴 |
| `AssumeRole` | 一時Credential取得 |
| `metrics / logs` | 観測Data |
| `control API` | 構成変更 |

`A → B`だけでは、Request、Replication、Permissionのどれか分からない。

---

# 3. Web request path

```mermaid
flowchart LR
    U[User]
    D[Route 53]
    C[CloudFront]
    A[ALB]
    E[ECS Service]
    R[(Aurora)]
    K[(ElastiCache)]

    U -->|DNS query| D
    D -->|Alias answer| U
    U -->|HTTPS request| C
    C -->|dynamic request| A
    A -->|forwarded HTTP| E
    E -->|session read/write| K
    E -->|SQL transaction| R
```

## この図が答える問い

- 名前解決と通信を分けられるか
- Edge、Entry、Compute、Stateを分けられるか
- SessionとTransactionの正本を指せるか

## この図に入れないもの

- Backup
- CI/CD
- Organizations

別の問いなので別図にする。

---

# 4. Event-driven processing

```mermaid
flowchart LR
    P[Producer]
    E[EventBridge]
    Q[SQS]
    W[Worker]
    D[(Database)]
    X[DLQ]

    P -->|business event| E
    E -->|route durable work| Q
    Q -->|receive message| W
    W -->|update result| D
    W -->|repeated failure| X
```

## 読むポイント

- EventBridgeはEventの振り分け
- SQSは未処理Workの保持
- Workerは処理主体
- DLQは失敗の終点ではなく調査入口

---

# 5. Workflow and compensation

```mermaid
stateDiagram-v2
    [*] --> OrderReceived
    OrderReceived --> Payment
    Payment --> ReserveInventory: success
    Payment --> Failed: failure
    ReserveInventory --> ArrangeShipping: success
    ReserveInventory --> RefundPayment: failure
    RefundPayment --> Failed
    ArrangeShipping --> Completed
    Completed --> [*]
    Failed --> [*]
```

State machineは、単なるサービス接続図より、分岐と補償を表すのに向く。

---

# 6. Multi-account operating model

```mermaid
flowchart TB
    ORG[AWS Organizations]
    SEC[Security Account]
    LOG[Log Archive Account]
    NET[Network Account]
    PROD[Production Account]
    DEV[Development Account]

    ORG --> SEC
    ORG --> LOG
    ORG --> NET
    ORG --> PROD
    ORG --> DEV
    NET -.shared connectivity.-> PROD
    NET -.shared connectivity.-> DEV
    PROD -->|organization logs| LOG
    DEV -->|organization logs| LOG
    SEC -.security findings.-> PROD
    SEC -.security findings.-> DEV
```

## 境界を示す

Account境界は、権限、請求、Quota、Blast radiusの境界である。

図では、Accountを箱として描くか、名前へ`Account`を含める。

---

# 7. Human access and workload access

```mermaid
flowchart LR
    H[Employee]
    IDP[Corporate IdP]
    IDC[IAM Identity Center]
    PS[Permission Set]
    ROLE[IAM Role in Account]
    APP[ECS Task / Lambda]
    TR[Task Role / Execution Role]
    AWS[AWS API]

    H -->|authenticate| IDP
    IDP -->|federation| IDC
    IDC -->|assign| PS
    PS -->|provision role| ROLE
    ROLE -->|temporary credentials| AWS

    APP -->|assume workload role| TR
    TR -->|temporary credentials| AWS
```

HumanとWorkloadのCredential経路を同じ線で描かない。

---

# 8. Hybrid DNS

```mermaid
flowchart LR
    ON[On-prem Client]
    ODNS[On-prem DNS]
    IN[Resolver Inbound Endpoint]
    PHZ[Private Hosted Zone]
    EC2[Private AWS Resource]

    AWSR[AWS Workload]
    OUT[Resolver Outbound Endpoint]
    RULE[Forwarding Rule]
    ONZ[On-prem DNS Zone]

    ON -->|DNS query for aws.internal| ODNS
    ODNS -->|forward query| IN
    IN -->|lookup| PHZ
    PHZ -->|private answer| ON

    AWSR -->|DNS query for corp.internal| OUT
    OUT -->|conditional forwarding| RULE
    RULE -->|query| ONZ
```

## 重要

DNS queryが成功しても、DX/VPN/TGW RouteやSecurityが成立するとは限らない。

DNS図とNetwork path図は必要に応じて分ける。

---

# 9. Multi-AZ database failover

```mermaid
sequenceDiagram
    participant App as Application
    participant DNS as DB Endpoint
    participant P as Primary AZ-A
    participant S as Standby AZ-B

    App->>DNS: connect
    DNS->>P: current writer
    P->>S: synchronous replication
    Note over P: failure
    S-->>DNS: promoted
    App->>DNS: reconnect
    DNS->>S: new writer
```

この図は次を見せる。

- Replication
- Failure
- Promotion
- Client reconnection

「Multi-AZ」と箱を二つ置くだけでは、切替過程が分からない。

---

# 10. Region DR

```mermaid
flowchart LR
    U[Users]
    G[Route 53 / Global Accelerator]

    subgraph P[Primary Region]
      PA[Application]
      PD[(Primary Data)]
    end

    subgraph S[Secondary Region]
      SA[Standby Application]
      SD[(Secondary Data)]
    end

    U --> G
    G -->|normal traffic| PA
    G -.failover traffic.-> SA
    PD -->|cross-region replication| SD
```

## 図だけでは不足する情報

文章で補う。

- RTO / RPO
- Secondary capacity
- Write strategy
- KMS / Secret
- Failover trigger
- Failback
- Test frequency

図は全要件を詰め込むものではない。

---

# 11. Backup and restore

```mermaid
flowchart LR
    P[Production Account]
    V[Backup Vault]
    A[Backup Account Vault]
    T[Test Restore Environment]
    E[Recovery Evidence]

    P -->|scheduled backup| V
    V -->|cross-account copy| A
    A -->|restore| T
    T -->|technical + business validation| E
```

Backup取得とRestore検証を別の矢印で描く。

---

# 12. Deployment and rollback

```mermaid
flowchart LR
    U[Users]
    R[Traffic Router]
    B[Blue v1]
    G[Green v2]
    M[Metrics / Business check]

    U --> R
    R -->|current traffic| B
    R -.test traffic.-> G
    G --> M
    M -->|healthy: shift| R
    M -.unhealthy: rollback.-> B
```

文章で補う：

- Database compatibility
- Artifact version
- Configuration
- Rollback deadline
- Success metric

---

# 13. Observability

```mermaid
flowchart TD
    S[System]
    M[Metrics]
    L[Logs]
    T[Traces]
    A[Alarm / Analysis]
    R[Runbook / Automation]
    V[Verification]

    S --> M
    S --> L
    S --> T
    M --> A
    L --> A
    T --> A
    A --> R --> V
```

Metrics、Logs、Tracesを同じ箱にまとめず、役割を分ける。

---

# 14. Performance diagnosis

```mermaid
flowchart TD
    B[Business symptom]
    E[Entry]
    C[Compute]
    I[Integration]
    D[Data]
    X[External dependency]
    F[Bottleneck found]
    V[Load test and verify]

    B --> E
    E --> C
    C --> I
    C --> D
    C --> X
    E --> F
    C --> F
    I --> F
    D --> F
    X --> F
    F --> V
```

性能図では、サービス箱よりLatency内訳やQueue ageなどの数値を添えるとよい。

---

# 15. Migration timeline

```mermaid
flowchart LR
    A[Assess]
    W[Wave plan]
    L[Landing Zone]
    R[Replication]
    T[Test migration]
    C[Cutover]
    H[Hypercare]
    M[Modernize]

    A --> W --> L --> R --> T --> C --> H --> M
```

Migrationは構成図だけでなく、時間軸図が必要である。

---

# 16. Cutover sequence

```mermaid
sequenceDiagram
    participant Old as Old System
    participant Sync as Replication / CDC
    participant DNS as DNS / Router
    participant New as New System
    participant Biz as Business validation

    Old->>Sync: continuous changes
    Note over Old: freeze writes
    Old->>Sync: final delta
    Sync-->>New: caught up
    DNS->>New: switch traffic
    New->>Biz: smoke + business tests
    Biz-->>DNS: accept or rollback
```

Data divergenceが始まる地点を文章で明記する。

---

# 17. 図のレビュー項目

## Flow

- StartとEndが分かるか
- 矢印に意味があるか
- 同期 / 非同期が分かるか

## State

- Source of truthが分かるか
- Cache、Replica、Backupを区別したか

## Boundary

- Account、VPC、AZ、Region、Public/Privateが分かるか

## Failure

- 何が壊れるか
- 検知と切替を描いたか

## Responsibility

- User、AWS Service、Application、Operatorを区別したか

## Density

- 一枚一問になっているか
- 15箱を超えたら分割できないか
- Legendが必要なほど複雑になっていないか

---

# 18. 図を文章へ戻すテンプレート

```text
[主体] が [入力] を [宛先] へ送る。
[中間サービス] は [判断条件] を評価し、[処理] を行う。
状態の正本は [保存先] にある。
[障害] が起きた場合、[検知] により [切替 / Retry] が行われる。
ただし [制約 / 代償] がある。
```

図を見てこの文章が作れない場合、矢印または状態の意味が不足している。
