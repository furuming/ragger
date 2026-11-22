# 7. API一覧

Base pathは `/api/v1`、通常の入出力はJSON、ファイル登録のみ `multipart/form-data` とする。日時はUTCのISO 8601形式で返し、画面側でローカル表示へ変換する。IDはUUID文字列とする。

## 7.1 エンドポイント

| Method | Path | 概要 | 主な成功応答 |
|---|---|---|---|
| `POST` | `/documents` | 文書アップロードと取り込み予約 | `202 Accepted` |
| `GET` | `/documents` | 文書一覧取得 | `200 OK` |
| `GET` | `/documents/{document_id}` | 文書詳細取得 | `200 OK` |
| `DELETE` | `/documents/{document_id}` | 文書と関連Chunkの削除 | `204 No Content` |
| `POST` | `/documents/{document_id}/reindex` | 文書の再取り込み予約 | `202 Accepted` |
| `POST` | `/chat` | 質問回答 | `200 OK` |
| `GET` | `/health` | システム状態取得 | `200 OK` / `503 Service Unavailable` |

## 7.2 主要Schema

### 文書登録

Request:

```text
Content-Type: multipart/form-data
file: UploadFile (必須、PDF/Markdown/TXT)
```

Response (`202`):

```json
{
  "id": "document-uuid",
  "filename": "travel_policy.pdf",
  "file_type": "pdf",
  "status": "pending",
  "chunk_count": 0,
  "error_message": null,
  "created_at": "2026-11-22T01:00:00Z",
  "updated_at": "2026-11-22T01:00:00Z"
}
```

同一SHA-256の文書が既に登録されている場合は `409 Conflict` とし、既存文書IDをエラー詳細へ含める。サーバーは表示用の元ファイル名と、パストラバーサルを防ぐ実保存名を分離する。

### 文書一覧

`GET /documents?limit=50&offset=0&status=completed`

```json
{
  "items": [],
  "total": 0,
  "limit": 50,
  "offset": 0
}
```

- `status` は任意。指定時は状態で絞り込む。
- 並び順は `created_at DESC` を既定とする。
- Detailは登録応答の項目に加え、必要に応じて抽出テキストのプレビューを返す。抽出テキストを永続化しない構成では省略可能とする。

### Chat

Request:

```json
{
  "question": "出張時の宿泊費上限は？",
  "top_k": 5
}
```

Response:

```json
{
  "answer": "規程では、宿泊費の上限は…です。",
  "sources": [
    {
      "document_id": "document-uuid",
      "chunk_id": "chunk-uuid",
      "filename": "travel_policy.pdf",
      "page": 3,
      "section": null,
      "text": "第5条 宿泊費…",
      "score": 0.87
    }
  ],
  "model": "configured-model"
}
```

- `question` は空白のみを禁止し、最大長を設定値またはSchemaで制限する。
- `top_k` は任意で、未指定時は設定値を利用する。過大な値を防ぐ上限を設ける。
- 検索結果がない場合も `200` とし、回答不能を明示した `answer` と空の `sources` を返す。
- `sources` はLLMに生成させず、実際の検索結果からBackendが構築する。

### Health

```json
{
  "status": "healthy",
  "components": {
    "postgresql": { "status": "healthy" },
    "qdrant": { "status": "healthy" },
    "ollama": { "status": "healthy" }
  }
}
```

Backend自身が応答でき、必須依存先も利用可能なら `200`、必須依存先が利用不能なら `503` を返す。レスポンスには秘密情報、内部ファイルパス、認証情報を含めない。

## 7.3 共通エラー形式

```json
{
  "error": {
    "code": "DUPLICATE_DOCUMENT",
    "message": "同じ内容の文書が既に登録されています。",
    "details": {
      "document_id": "existing-document-uuid"
    },
    "request_id": "request-uuid"
  }
}
```

`code` はFrontendで分岐可能な安定した識別子、`message` はユーザー向け日本語、`details` は安全な補足情報とする。
