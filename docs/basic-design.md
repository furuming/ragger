# ローカルRAG基盤 基本設計書

本書はMVP（Phase 1〜5）を対象とする基本設計書の目次です。本文は章ごとに `docs/basic-design/` 配下へ分割しています。

| 章 | 内容 |
|---|---|
| [0. 文書情報](basic-design/00-document-information.md) | 文書メタデータと設計上の前提 |
| [1. システム概要](basic-design/01-system-overview.md) | 目的、対象範囲、対象外 |
| [2. 技術スタック](basic-design/02-technology-stack.md) | 採用技術と用途 |
| [3. アーキテクチャ](basic-design/03-architecture.md) | システム構成、配置構成、Backendレイヤー |
| [4. コンポーネント責務](basic-design/04-component-responsibilities.md) | 各コンポーネントの責務と責務外 |
| [5. 画面一覧](basic-design/05-screens.md) | 画面、Route、共通UI方針 |
| [6. 機能一覧](basic-design/06-features.md) | 機能ID、概要、実装Phase |
| [7. API一覧](basic-design/07-api.md) | REST API、Schema、共通エラー形式 |
| [8. DB設計](basic-design/08-database.md) | PostgreSQL、Qdrant、ファイル保存 |
| [9. 文書取り込みフロー](basic-design/09-document-ingestion.md) | 登録、再取り込み、削除フロー |
| [10. 質問回答フロー](basic-design/10-question-answering.md) | RAG処理とContext・Prompt規則 |
| [11. LangGraph設計](basic-design/11-langgraph.md) | Graph、State、Node責務 |
| [12. エラーハンドリング](basic-design/12-error-handling.md) | HTTPエラー、ログ、回復方針 |
| [13. 非機能要件](basic-design/13-non-functional-requirements.md) | セキュリティ、性能、信頼性、保守性など |
| [14. MVP受け入れ条件との対応](basic-design/14-mvp-acceptance-criteria.md) | MVP完了時のスモークテスト |

要件定義は [design.md](design.md) を参照してください。
