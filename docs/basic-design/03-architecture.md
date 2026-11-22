# 3. アーキテクチャ

## 3.1 システム構成

```mermaid
flowchart TB
    U[Browser]
    FE[Nuxt Frontend]
    API[FastAPI REST API]
    DS[Document Service]
    CS[Chat Service]
    IP[Ingestion Service]
    LG[LangGraph]
    RS[Retriever Service]
    GS[Generator Service]
    ES[Embedding Service]
    PARSER[Document Parser]
    CHUNKER[Chunker]
    REPO[Document Repository]
    FS[(Local File System)]
    DB[(PostgreSQL)]
    QD[(Qdrant)]
    OL[Ollama]

    U --> FE
    FE -->|HTTP/JSON・multipart| API
    API --> DS
    API --> CS
    DS --> IP
    DS --> REPO
    REPO --> DB
    DS --> FS
    IP --> PARSER
    IP --> CHUNKER
    IP --> ES
    ES --> OL
    IP --> QD
    CS --> LG
    LG --> RS
    LG --> GS
    RS --> ES
    RS --> QD
    GS --> OL
```

## 3.2 配置構成

```mermaid
flowchart LR
    subgraph LocalMachine[ローカルPC]
        B[Browser]
        O[Ollama<br/>ホスト起動を標準]
        subgraph Compose[Docker Compose]
            F[frontend]
            A[backend]
            P[postgres]
            Q[qdrant]
            V1[(uploads volume)]
            V2[(postgres volume)]
            V3[(qdrant volume)]
        end
    end

    B -->|localhost| F
    F -->|/api/v1| A
    A --> P
    A --> Q
    A --> O
    A --> V1
    P --> V2
    Q --> V3
```

- 公開ポートは原則としてループバックアドレスへバインドする。
- FrontendからBackendへの接続先、BackendからQdrant・Ollamaへの接続先は環境変数で切り替える。
- APIはステートレスに保ち、永続データはPostgreSQL、Qdrant、ローカルファイルに分離する。
- MVPでは不必要なマイクロサービス化を避け、Backend内を責務別モジュールとして分割する。

## 3.3 Backendレイヤー

依存方向は外側から内側へ向ける。

```text
API Routes / Graph Nodes
          ↓
Application Services
          ↓
Repository・Gateway Interfaces
          ↓
PostgreSQL・Qdrant・Ollama・File System Adapters
```

LangGraph Nodeは外部システムへ直接アクセスせず、Retriever ServiceまたはGenerator Serviceを呼び出す。FastAPI Routeもビジネス処理を持たず、入力検証、Service呼び出し、HTTPレスポンス変換に限定する。
