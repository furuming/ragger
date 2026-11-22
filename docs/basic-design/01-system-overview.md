# 1. システム概要

## 1.1 目的

ユーザーがブラウザから社内文書を登録し、その文書だけを根拠として日本語で質問回答できるローカルRAG Webアプリケーションを提供する。回答には、根拠となった文書名、ページ番号、該当テキストなどの参照元を付与する。

## 1.2 対象範囲

MVPでは次を提供する。

- PDF、Markdown、TXT文書の登録、一覧、詳細、削除、再取り込み
- 文書の解析、正規化、Chunk分割、Embedding生成、Qdrant登録
- Dense Vector Searchによる関連Chunk検索
- LangGraphで制御する検索・回答生成フロー
- Ollama上のLLMによる日本語回答生成
- 回答根拠となる参照文書・Chunkの表示
- ローカルPC上でのDocker Composeによる実行

## 1.3 対象外

- 認証・認可、ユーザー・組織管理
- 外部LLM APIの利用
- Word、Excel、HTML、画像、OCRの取り込み
- Hybrid Search、BM25、Reranking、Query Rewrite、Adaptive RAG、Multi-Agent、GraphRAG
- 検索精度の評価基盤
- チャット履歴の永続化、ストリーミング応答
- 複数Backendプロセスを前提とする分散ジョブ処理
- インターネットへの公開、本番向け高可用性構成
