# 12. エラーハンドリング

## 12.1 HTTPエラー方針

| ケース | HTTP | Error code | ユーザー向け対応 |
|---|---:|---|---|
| 未対応形式 | 415 | `UNSUPPORTED_FILE_TYPE` | 対応形式を案内 |
| サイズ超過 | 413 | `FILE_TOO_LARGE` | 上限値を案内 |
| 重複文書 | 409 | `DUPLICATE_DOCUMENT` | 既存文書を案内 |
| 文書なし | 404 | `DOCUMENT_NOT_FOUND` | 一覧へ戻す |
| 取り込み中の競合操作 | 409 | `DOCUMENT_BUSY` | 完了後の再試行を案内 |
| 入力不正 | 422 | `VALIDATION_ERROR` | 対象フィールドを案内 |
| Qdrant利用不能 | 503 | `VECTOR_STORE_UNAVAILABLE` | 起動状態の確認を案内 |
| Ollama利用不能 | 503 | `OLLAMA_UNAVAILABLE` | 起動状態の確認を案内 |
| モデル未取得 | 503 | `MODEL_UNAVAILABLE` | 必要モデルを案内 |
| 外部処理タイムアウト | 504 | `UPSTREAM_TIMEOUT` | 時間をおいて再試行 |
| 予期しない障害 | 500 | `INTERNAL_ERROR` | request_idを表示 |

検索結果なしは障害ではないため `200` とする。取り込みパイプライン内のPDF解析、Embedding、Qdrant登録失敗は非同期処理後に `status=failed` と `error_message` で通知する。

## 12.2 ログ方針

- 構造化ログを標準出力へ出し、時刻、level、request_id、処理名、document_id、所要時間、エラー種別を含める。
- 質問全文、Chunk本文、アップロード文書本文は既定でログへ出さない。
- 例外の詳細とスタックトレースはBackendログへ、Frontendには安全なメッセージだけを返す。
- OllamaやQdrant呼び出しには接続・読み取りタイムアウトを設定する。
- 一時的な接続失敗に対する再試行は回数と待機時間に上限を設け、入力不正やモデル未取得は再試行しない。

## 12.3 整合性・回復

- 取り込み失敗時は、部分登録されたQdrant Pointを文書IDで削除する。
- 再取り込み可能なように元ファイルとPostgreSQLレコードを保持する。
- 削除失敗時は残存箇所をログへ記録し、同じ削除要求を再実行しても安全な冪等処理にする。
- 起動時にDB Migration、保存ディレクトリ、Qdrant CollectionとEmbedding次元の整合性を検証する。
