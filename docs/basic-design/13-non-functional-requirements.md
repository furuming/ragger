# 13. 非機能要件

## 13.1 セキュリティ・プライバシー

- すべての文書、Embedding、Prompt、回答をローカル環境外へ送信しない。
- サービスのポートは `127.0.0.1` を既定とし、LAN・インターネットへ不用意に公開しない。
- 認証がないため、ローカルPC外から到達可能な構成は禁止する。
- ファイル名のサニタイズ、UUID保存パス、許可リスト方式の拡張子・MIME検証によりパストラバーサルを防ぐ。
- アップロードサイズ、質問長、`top_k` に上限を設け、過剰なリソース消費を防ぐ。
- 文書内のPrompt Injectionを命令として扱わず、Contextを信頼できないデータとして隔離する。
- `.env`、アップロード文書、PostgreSQLの永続化データ、Qdrantデータ、ログをGitへ含めない。

## 13.2 性能・容量

- 文書登録APIは保存と受付完了後に `202` を返し、長時間の取り込み処理でHTTP接続を保持しない。
- Chat APIは非ストリーミングとし、Ollamaのモデル・ハードウェア性能に応じたタイムアウトを設定可能にする。
- Chunk Size、Overlap、Embeddingバッチサイズ、`top_k`、最大ファイルサイズを環境変数で変更可能にする。
- 個人利用・単一同時利用をMVPの主対象とする。具体的な応答時間SLOはハードウェア確認後に設定する。

## 13.3 信頼性・データ整合性

- UUIDとSHA-256 UNIQUE制約で文書を識別し、重複登録を防止する。
- 決定的Chunk IDと文書単位削除により再取り込み時の重複を防止する。
- PostgreSQL、Qdrant、uploadsはDocker Volumeまたはホストマウントで永続化する。
- 非同期処理失敗を `failed` として可視化し、元ファイルから再試行できるようにする。
- MVPでは高可用性や完全な分散トランザクションを保証しない。中間失敗は補償処理と再実行で回復する。

## 13.4 保守性・拡張性

- API、Application Service、Repository/Gateway、外部Adapterを分離する。
- RetrieverをInterface化し、将来のHybrid SearchやReranking追加時もChat APIとGraphの変更を局所化する。
- 設定はPydantic Settings相当で一元管理し、環境変数名と既定値を `.env.example` に記載する。
- Python、Node.js双方の依存をロックし、コンテナイメージもバージョン固定する。
- DB Schema変更はMigrationで管理し、手作業によるDB更新を避ける。

## 13.5 テスト容易性

- Ollama、Qdrant、File SystemはAdapterとして差し替え可能にする。
- Chunker、Parser、Repositoryを単体テストする。
- Document/Chat APIの正常系・異常系をテストする。
- Fake Serviceを利用してLangGraphのNode順序とState更新を確認する。
- 架空のPDF、Markdown、TXTを用いたE2Eスモークテストを用意し、実社内文書をテスト資産にしない。

## 13.6 可観測性

- `/api/v1/health` でPostgreSQL、Qdrant、Ollamaの個別状態を確認できる。
- request_idでFrontend表示とBackendログを対応付ける。
- 文書の状態、Chunk数、最終エラーをUIとAPIから確認できる。
- MVPではメトリクス基盤や分散トレーシングは導入しない。

## 13.7 利用性・互換性

- 最新の主要PCブラウザを対象とし、モバイル最適化は必須としない。
- ユーザー向け文言と生成回答は原則として日本語とする。
- Windows/macOS/LinuxでOllamaの配置が異なることを考慮し、接続URLを設定可能にする。
- CPUのみでも起動できる構成を維持し、モデルサイズと必要メモリは環境に合わせて変更可能にする。
