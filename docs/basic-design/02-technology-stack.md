# 2. 技術スタック

| レイヤー | 採用技術 | 用途 |
|---|---|---|
| Frontend | Nuxt / Vue / TypeScript | SPA相当の画面、APIクライアント、画面内状態管理 |
| Backend | Python / FastAPI | REST API、ユースケース制御 |
| Validation | Pydantic | API入出力、設定値の検証 |
| Workflow | LangGraph | 質問回答フローのオーケストレーション |
| LLM Runtime | Ollama | ローカルでのGeneration、Embedding |
| Vector DB | Qdrant | Chunkベクトルと検索用Payloadの保存 |
| Metadata DB | PostgreSQL | 文書メタデータと取り込み状態の保存 |
| Document Parser | PDF用ライブラリ、標準テキスト処理 | PDF、Markdown、TXTのテキスト抽出 |
| Infrastructure | Docker Compose | Frontend、Backend、Qdrantの起動と永続化 |
| Package Management | uv / npm | Python / Node.js依存管理 |
| Test | pytest / FastAPI TestClient / Frontend test tools | 単体、API、スモークテスト |

ライブラリとコンテナイメージは実装開始時点の互換性がある安定版を選び、ロックファイルまたは明示的なバージョンで固定する。`latest` タグは再現性がないため、確定後は使用しない。
