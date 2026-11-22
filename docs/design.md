# ローカルRAG基盤の設計・実装

あなたはシニアソフトウェアエンジニア兼ソフトウェアアーキテクトです。

以下の要件に基づき、ローカル環境で動作するRAG（Retrieval-Augmented Generation）Webアプリケーションを設計・実装してください。

本プロジェクトは個人開発であり、3〜4ヶ月かけて段階的に構築することを想定しています。

最優先事項は、**社内文書を登録し、その内容に基づいてLLMが回答するシステムを完成させること**です。

検索品質の定量評価や高度なRAG手法は、今回の実装対象から除外してください。

---

# 1. プロジェクト概要

## 1.1 目的

ローカル環境で動作するRAG基盤を構築する。

ユーザーはブラウザから文書をアップロードし、自然言語による質問を行える。

システムは登録された文書を検索し、関連する内容をOllamaで動作するLLMに渡して回答を生成する。

回答には参照元文書の情報を付与する。

## 1.2 開発方針

以下の原則を守ること。

1. MVPを優先する。
2. 過剰な抽象化を避ける。
3. 各モジュールの責務を明確にする。
4. 将来的な機能拡張を妨げないクリーンアーキテクチャに準ずる設計にする。
5. LangGraphはワークフロー制御に限定する。
6. 外部LLM APIは使用しない。
7. ローカル環境だけで動作する構成にする。
8. 不必要なマイクロサービス化をしない。
9. 認証・認可は実装しない。
10. 評価基盤は実装しない。

単にコードを生成するのではなく、実際に起動・動作確認できるアプリケーションを作成すること。

---

# 2. 技術スタック

以下を基本構成とする。

| Layer | Technology |
|---|---|
| Frontend | Nuxt.js / Vue / TypeScript |
| Backend | Python / FastAPI |
| Validation | Pydantic |
| Workflow | LangGraph |
| LLM Runtime | Ollama |
| Generation | Ollama |
| Embedding | Ollama |
| Vector Database | Qdrant |
| Metadata Database | PostgreSQL |
| Infrastructure | Docker Compose |
| Package Management | uv / npm |

バージョンは実装時点で互換性のある安定版を選択すること。

依存ライブラリは明示的に管理すること。

---

# 3. システムアーキテクチャ

以下の構成を基本とする。

```text
                       Browser
                          |
                          v
                   Nuxt.js Frontend
                          |
                          v
                     FastAPI
                          |
              +-----------+-----------+
              |                       |
              v                       v
        Document Service         Chat Service
              |                       |
              v                       v
       Ingestion Pipeline          LangGraph
              |                       |
              v                       v
         Document Parser        Retrieve Node
              |                       |
              v                       v
            Chunker              Qdrant Search
              |                       |
              v                       v
       Ollama Embedding          Generate Node
              |                       |
              v                       v
            Qdrant               Ollama LLM
                                      |
                                      v
                                Answer + Sources
```

Metadata DatabaseにはPostgreSQLを使用する。

文書ファイルはローカルファイルシステムに保存する。

---

# 4. 実装対象機能

## 4.1 文書管理

以下を実装する。

- 文書アップロード
- 文書一覧取得
- 文書詳細取得
- 文書削除
- 文書再取り込み
- 文書取り込み状態管理

対象ファイル形式：

- PDF
- Markdown
- TXT

初期段階では以下を対象外とする。

- Word
- Excel
- HTML
- OCR
- 画像解析

### 文書登録処理

```text
Upload
  ↓
Save File
  ↓
Register Metadata
  ↓
Parse Document
  ↓
Normalize Text
  ↓
Split into Chunks
  ↓
Generate Embeddings
  ↓
Upsert to Qdrant
  ↓
Update Document Status
```

以下の条件を満たすこと。

- 文書IDにはUUIDを使用する。
- SHA-256による重複検出を行う。
- Chunkには元文書IDを保持する。
- PDFの場合はページ番号を保持する。
- 失敗した取り込みを再試行できるようにする。
- 再取り込み時に旧Chunkが重複しないようにする。
- 文書削除時はQdrant上の関連Chunkも削除する。
- ファイル形式・サイズを検証する。
- 取り込み失敗時に原因を確認できるようにする。

---

# 5. RAG処理

## 5.1 初期RAG構成

初期段階では以下のシンプルなRAGを実装する。

```text
User Question
      |
      v
Query Embedding
      |
      v
Qdrant Dense Search
      |
      v
Retrieve Top-K Chunks
      |
      v
Build Context
      |
      v
Ollama Generation
      |
      v
Answer + Sources
```

初期実装ではDense Vector Searchのみ使用する。

以下は実装しない。

- Hybrid Search
- BM25
- Reranking
- Query Rewrite
- Adaptive RAG
- Multi-Agent
- GraphRAG

ただし、将来的にRetrieverを差し替えられるように責務を分離する。

## 5.2 回答生成

以下の条件を満たすこと。

- 検索されたChunkのみを根拠として回答する。
- 根拠がない場合は回答できないことを明示する。
- 回答と参照元文書情報を返す。
- 原則として日本語で回答する。
- Contextには文書名やページ番号を含める。
- 文書内に含まれる指示文をシステム命令として扱わない。

初期段階では非ストリーミング応答でよい。

---

# 6. LangGraph設計

LangGraphは質問処理のオーケストレーションに使用する。

初期Graphは以下とする。

```text
START
  |
  v
retrieve
  |
  v
generate
  |
  v
END
```

以下のNodeを作成する。

### retrieve_node

責務：

- Queryを受け取る
- Retriever Serviceを呼び出す
- 検索結果をStateに格納する

### generate_node

責務：

- 検索結果を取得する
- Contextを構築する
- Generator Serviceを呼び出す
- 回答をStateに格納する

検索結果が存在しない場合は、根拠がない旨を返す。

Stateは必要最低限にする。

```python
from typing import TypedDict


class Source(TypedDict):
    document_id: str
    chunk_id: str
    filename: str
    text: str
    score: float


class RAGState(TypedDict):
    question: str
    sources: list[Source]
    answer: str
```

LangGraph NodeにQdrantやOllamaへのアクセス処理を直接実装しないこと。

各NodeはServiceを呼び出す薄い構成とする。

---

# 7. データベース設計

## 7.1 PostgreSQL

文書メタデータを管理する。

documentsテーブルを作成する。

| Column | Type | Description |
|---|---|---|
| id | UUID PRIMARY KEY | 文書UUID |
| filename | VARCHAR(255) | 元ファイル名 |
| file_path | TEXT | 保存先 |
| file_type | VARCHAR(16) | ファイル形式 |
| content_hash | CHAR(64) | SHA-256 |
| status | VARCHAR(16) | 取り込み状態 |
| chunk_count | INTEGER | Chunk数 |
| error_message | TEXT NULL | エラー情報 |
| created_at | TIMESTAMPTZ | 作成日時 |
| updated_at | TIMESTAMPTZ | 更新日時 |

取り込み状態は以下とする。

```text
pending
processing
completed
failed
```

必要なIndexや制約を設定すること。

## 7.2 Qdrant

Collection名：

```text
document_chunks
```

1 Point = 1 Chunkとする。

Payload例：

```json
{
  "document_id": "uuid",
  "chunk_index": 0,
  "filename": "example.pdf",
  "text": "document content...",
  "page": 1,
  "section": null
}
```

Vectorの次元数はEmbeddingモデルに合わせる。

EmbeddingモデルとCollectionの互換性を確認すること。

Chunk IDは再取り込み時にも安定する方式で生成すること。

---

# 8. Backend API

FastAPIで以下のREST APIを実装する。

Base path：

```text
/api/v1
```

## Documents

```http
POST /api/v1/documents
GET /api/v1/documents
GET /api/v1/documents/{document_id}
DELETE /api/v1/documents/{document_id}
POST /api/v1/documents/{document_id}/reindex
```

POSTはファイルアップロードを受け付ける。

取り込み処理が長時間になる場合を考慮し、登録直後はpendingまたはprocessingを返す方式とする。

フロントエンドは文書一覧APIで状態を確認できるようにする。

## Chat

```http
POST /api/v1/chat
```

Request：

```json
{
  "question": "出張時の宿泊費上限は？",
  "top_k": 5
}
```

Response：

```json
{
  "answer": "規程第5条に記載されています。",
  "sources": [
    {
      "document_id": "uuid",
      "chunk_id": "uuid",
      "filename": "travel_policy.pdf",
      "page": 3,
      "text": "第5条 宿泊費...",
      "score": 0.87
    }
  ],
  "model": "configured-model"
}
```

## Health

```http
GET /api/v1/health
```

システムの動作状態を確認できるようにする。

## Settings

初期実装では設定ファイルによる管理を基本とする。

以下は設定ファイルから変更できるようにする。

- LLMモデル
- Embeddingモデル
- top_k
- Chunk Size
- Chunk Overlap
- Ollama URL
- Qdrant URL

Settings APIおよび画面からの設定変更はMVP完成後の拡張として扱ってよい。

---

# 9. Frontend設計

Nuxt.js + TypeScriptで実装する。

初期段階では以下の画面を作成する。

## 9.1 Chat画面

機能：

- 質問入力
- 質問送信
- 回答表示
- 参照文書表示
- 参照Chunkの展開
- Loading表示
- エラー表示

初期段階ではチャット履歴の永続化は不要とする。

画面内の状態管理のみでよい。

## 9.2 Documents画面

機能：

- 文書アップロード
- 文書一覧
- 取り込み状態表示
- 文書詳細表示
- 文書削除
- 再取り込み

文書取り込み中は状態がわかるUIにする。

## 9.3 Document Detail画面

表示項目：

- 文書名
- 文書形式
- 登録日時
- 取り込み状態
- Chunk数
- エラー情報

取得可能であれば抽出したテキストも表示する。

## 9.4 Settings

初期段階では設定値の参照のみでもよい。

設定変更UIは後続フェーズで実装してよい。

## UI設計方針

- シンプルで見やすいデザイン
- PCブラウザを主対象とする
- 不必要なアニメーションを実装しない
- API通信中はLoadingを表示する
- エラー時はユーザーに原因を示す
- API型定義を明確にする
- コンポーネントを適切に分割する

---

# 10. ディレクトリ構成

以下を基本とする。

```text
rag-platform/
├── frontend/
│   ├── src/
│   │   ├── app/
│   │   │   ├── chat/
│   │   │   ├── documents/
│   │   │   └── settings/
│   │   ├── components/
│   │   ├── features/
│   │   └── lib/
│   └── package.json
│
├── backend/
│   ├── app/
│   │   ├── api/
│   │   │   └── routes/
│   │   ├── graph/
│   │   │   ├── state.py
│   │   │   ├── nodes.py
│   │   │   └── graph.py
│   │   ├── services/
│   │   │   ├── ingestion/
│   │   │   ├── retrieval/
│   │   │   ├── embedding/
│   │   │   └── generation/
│   │   ├── repositories/
│   │   ├── models/
│   │   ├── schemas/
│   │   └── core/
│   ├── tests/
│   └── pyproject.toml
│
├── configs/
├── data/
│   ├── uploads/
│   ├── postgres/
│   └── qdrant/
├── compose.yaml
├── Makefile
├── .env.example
├── .gitignore
└── README.md
```

必要に応じて構成を調整してよいが、変更理由を説明すること。

---

# 11. インフラ構成

Docker Composeを使用する。

構成：

```text
frontend
backend
postgres
qdrant
ollama
```

OllamaについてはGPU環境やOSの違いを考慮し、ホスト環境での起動も許容する。

以下を満たすこと。

- PostgreSQLを永続化する。
- Qdrantを永続化する。
- アップロードファイルを永続化する。
- サービス間の接続先は環境変数で管理する。
- ローカルホスト以外へ不用意にポートを公開しない。
- 外部LLM APIを呼び出さない。
- シークレットや社内文書をGitに含めない。

認証は実装しないが、ローカル限定の構成とする。

---

# 12. エラーハンドリング

最低限、以下のケースを考慮すること。

### 文書登録

- 未対応のファイル形式
- ファイルサイズ超過
- 重複ファイル
- PDF解析失敗
- Embedding生成失敗
- Qdrant登録失敗
- 再取り込み失敗

### 質問処理

- Ollama未起動
- Qdrant未起動
- Embeddingモデル未取得
- LLMモデル未取得
- 検索結果なし
- LLM生成失敗
- タイムアウト

ユーザーに表示するエラーと、開発者向けログを分離すること。

---

# 13. 開発フェーズ

4ヶ月程度の個人開発を想定する。

ただし、今回の依頼では4ヶ月分を一度に実装しないこと。

まずは設計と実装計画を完成させ、その後段階的に実装する。

## Phase 1：基盤構築

対象：

- Repository
- Docker Compose
- FastAPI
- Nuxt.js
- Ollama接続
- Qdrant接続
- LangGraphの最小構成

完了条件：

各サービスが起動し、接続を確認できる。

## Phase 2：Minimum RAG

対象：

- Markdown / TXT取り込み
- Chunking
- Embedding
- Qdrant登録
- Dense Retrieval
- Ollama回答生成
- Chat API

完了条件：

サンプル文書に対して質問し、回答と出典を取得できる。

## Phase 3：Frontend

対象：

- Chat画面
- Documents画面
- API接続
- Loading / Error処理

完了条件：

ブラウザから文書登録と質問ができる。

## Phase 4：文書管理機能

対象：

- PDF対応
- PostgreSQL文書管理
- 文書一覧
- 文書削除
- 再取り込み
- 取り込み状態管理

完了条件：

複数文書を継続的に管理できる。

## Phase 5：安定化

対象：

- エラーハンドリング
- ログ
- 単体テスト
- APIテスト
- README
- 起動手順整理

完了条件：

新しいローカル環境でも手順に従って起動できる。

---

# 14. テスト方針

検索精度を測定する評価基盤は作成しない。

ただし、通常のソフトウェアテストは実装すること。

最低限：

- Chunkerの単体テスト
- Document Parserの単体テスト
- Repositoryの単体テスト
- APIの正常系・異常系テスト
- LangGraphのフロー確認
- サンプル文書を使用したE2Eスモークテスト

実際の社内文書はテストリポジトリに含めない。

テストには架空のサンプル文書を使用すること。

---

# 15. 完了条件

以下をMVPのDefinition of Doneとする。

- [ ] Docker Composeで必要なサービスが起動する
- [ ] Ollamaと接続できる
- [ ] Qdrantと接続できる
- [ ] ブラウザから文書をアップロードできる
- [ ] PDF / Markdown / TXTを取り込める
- [ ] 文書をChunkに分割できる
- [ ] Embeddingを生成できる
- [ ] QdrantへChunkを登録できる
- [ ] 文書一覧を表示できる
- [ ] 文書を削除できる
- [ ] 文書を再取り込みできる
- [ ] ブラウザから質問できる
- [ ] LangGraph経由でRetrievalとGenerationを実行できる
- [ ] Ollamaが回答を生成できる
- [ ] 回答に参照元文書が表示される
- [ ] 検索結果がない場合に適切な応答が返る
- [ ] 主要なエラーが処理される
- [ ] READMEに環境構築手順がある

---

# 16. 今回の作業指示

まず、既存Repositoryを確認してください。

既存のコードや設定ファイルがある場合は、内容を把握し、利用可能なものを再利用してください。

既存の実装を理由なく削除・置換しないでください。

そのうえで、以下の順番で作業してください。

## Step 1：基本設計書作成

`docs/basic-design.md`を作成してください。

記載内容：

1. システム概要
2. 技術スタック
3. アーキテクチャ
4. コンポーネント責務
5. 画面一覧
6. 機能一覧
7. API一覧
8. DB設計
9. 文書取り込みフロー
10. 質問回答フロー
11. LangGraph設計
12. エラーハンドリング
13. 非機能要件

Mermaidで主要な構成図とシーケンス図を作成してください。

## Step 2：実装計画作成

`docs/implementation-plan.md`を作成してください。

Phase 1〜5を具体的なタスクに分解してください。

各タスクには以下を記載してください。

- Task ID
- 作業内容
- 対象ファイル
- 依存タスク
- 完了条件
- 想定工数

依存関係を考慮して実装順序を決定してください。

## Step 3：技術的な判断事項

`docs/technical-decisions.md`を作成してください。

以下について採用方針と理由を記載してください。

- OllamaのGenerationモデル
- OllamaのEmbeddingモデル
- Qdrant Collection設計
- PDF Parser
- Chunking方式
- PostgreSQLアクセス方式
- 文書取り込みの非同期処理方式
- FrontendとBackendの通信方式
- LangGraph State設計

モデル選定では日本語対応とローカル環境での実行負荷を考慮してください。

ハードウェアスペックが不明なため、モデル名・サイズ・必要メモリは設定変更可能にしてください。

## Step 4：Phase 1の実装

設計書と実装計画を作成した後、Phase 1のみ実装してください。

最低限、以下を作成してください。

- Frontendの初期プロジェクト
- Backendの初期プロジェクト
- Docker Compose
- 環境変数設定
- Qdrant接続処理
- Ollama接続処理
- LangGraph Skeleton
- Health API
- 起動用Makefile
- README

Phase 2以降の機能は実装しないでください。

## Step 5：動作確認と報告

可能な範囲で実際に起動し、動作確認してください。

確認項目：

- Backendが起動する
- Frontendが起動する
- Health APIが応答する
- Qdrantへの接続が成功する
- Ollamaへの接続が成功する
- LangGraphの最小Graphが実行できる

未導入のモデルやGPU不足など、環境依存で確認できない項目はその理由を記載してください。

最後に以下を報告してください。

1. 作成・変更したファイル
2. 実装した内容
3. 実行したテストと結果
4. 未確認事項
5. 次のPhaseで実装する内容

**今回はPhase 1までを実装対象とします。設計書はMVP全体を対象としてください。**

不明点は合理的なデフォルトを採用し、技術的判断事項に記録してください。

ただし、社内文書を外部サービスへ送信する設計にはしないでください。
