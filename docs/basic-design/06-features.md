# 6. 機能一覧

| ID | 機能 | 概要 | Phase |
|---|---|---|---|
| F-001 | システム接続確認 | Backend、Qdrant、Ollama等の状態を返す | 1 |
| F-002 | 最小LangGraph実行 | `retrieve`、`generate` Nodeの接続を検証する | 1 |
| F-101 | Markdown/TXT登録 | 保存、重複判定、非同期取り込み開始 | 2 |
| F-102 | テキスト抽出・正規化 | Markdown/TXTを読み込み検索用テキストに変換 | 2 |
| F-103 | Chunk分割 | 設定されたサイズとOverlapで分割 | 2 |
| F-104 | Embedding・登録 | Ollamaでベクトル化しQdrantへ登録 | 2 |
| F-105 | 質問回答 | Dense検索後にOllamaで回答とSourceを生成 | 2 |
| F-201 | Chat UI | ブラウザから質問し回答・Sourceを表示 | 3 |
| F-202 | Documents UI | ブラウザから登録し状態を確認 | 3 |
| F-301 | PDF登録 | ページ情報を保持してPDFを取り込む | 4 |
| F-302 | 文書一覧・詳細 | PostgreSQLの文書メタデータを取得 | 4 |
| F-303 | 文書削除 | File、PostgreSQL、Qdrant上の関連データを削除 | 4 |
| F-304 | 文書再取り込み | 既存Chunkを重複させず再生成 | 4 |
| F-305 | 取り込み状態管理 | pending / processing / completed / failedを管理 | 4 |
| F-401 | ログ・例外処理 | ユーザー向けエラーと開発者ログを分離 | 5 |
| F-402 | 自動テスト | Unit、API、Graph、E2Eスモークテスト | 5 |
| F-403 | セットアップ文書 | 新規環境で再現可能な起動手順 | 5 |
