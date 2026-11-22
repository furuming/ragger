# 14. MVP受け入れ条件との対応

MVP完了時には、以下を一連のスモークテストとして確認する。

1. Docker ComposeでFrontend、Backend、Qdrantが起動し、Ollamaへ接続できる。
2. ブラウザからPDF、Markdown、TXTを登録し、`completed` になることを確認できる。
3. 登録文書に関する質問へ、LangGraph経由で回答と実在するSourceが返る。
4. 根拠がない質問には回答不能であることを明示する。
5. 文書の削除によりFile、PostgreSQL、Qdrantの関連データが削除される。
6. 文書を再取り込みしてもChunkが重複しない。
7. 主要な異常が共通エラー形式または文書の `failed` 状態として確認できる。
8. READMEの手順だけで新しいローカル環境にセットアップできる。

