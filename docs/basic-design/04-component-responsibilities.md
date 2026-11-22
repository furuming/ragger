# 4. コンポーネント責務

| コンポーネント | 主な責務 | 責務外 |
|---|---|---|
| Nuxt Frontend | 画面表示、入力検証、API呼び出し、ローディング・エラー表示、状態ポーリング | RAG処理、文書解析 |
| FastAPI Routes | HTTP入出力、Schema検証、Service呼び出し、例外からHTTPへの変換 | DBや外部サービスへの直接アクセス |
| Document Service | 登録・取得・削除・再取り込みユースケース、重複判定、状態遷移 | PDF解析、Embedding生成の詳細 |
| Ingestion Service | 文書取り込みパイプラインの実行、失敗時の状態・エラー記録 | HTTPレスポンス生成 |
| Document Parser | ファイル形式別テキスト抽出、PDFページ番号の保持 | Chunk分割、保存 |
| Text Normalizer | 改行・空白・制御文字の正規化 | 文書内容の要約・翻訳 |
| Chunker | 正規化テキストの分割、ページ・節などのメタデータ継承 | ベクトル生成 |
| Embedding Service | Ollama Embedding API呼び出し、ベクトル次元の検証 | Qdrant検索条件の決定 |
| Document Repository | PostgreSQL上の文書メタデータCRUD、状態遷移 | Chunkベクトル管理 |
| Chunk Repository | Qdrant Collection作成、upsert、検索、文書単位削除 | Generation |
| Chat Service | 質問回答Graphの起動、結果整形 | Graph Node固有処理の直接実装 |
| LangGraph | `retrieve` → `generate` の順序とState受け渡し | Qdrant・Ollamaへの直接アクセス |
| Retriever Service | Query Embedding、Dense検索、Sourceへの変換 | 回答文生成 |
| Context Builder | 検索Chunkから出典付きContextを構築、長さ制御、文書内指示の隔離 | 検索 |
| Generator Service | System Prompt構築、Ollama Generation呼び出し、根拠なし応答 | 文書永続化 |
| Settings | 環境変数読込、既定値設定、起動時検証 | 画面からの動的設定変更 |
| Health Service | Backend、PostgreSQL、Qdrant、Ollamaの状態確認 | 障害の自動復旧 |
