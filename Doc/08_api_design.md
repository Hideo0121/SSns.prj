# API設計書

このドキュメントでは、スタッフ管理システムのAPI設計について詳細に説明します。

## API概要

- **ベースURL**: `/api`
- **認証方式**: Laravel Sanctum (Bearer Token)
- **データ形式**: JSON
- **文字エンコーディング**: UTF-8
- **HTTPステータスコード**: REST標準に準拠

## 認証

### 認証エンドポイント

#### POST /api/login
ユーザー認証を行い、アクセストークンを発行します。

**リクエスト:**
```json
{
  "user_code": "string (required)",
  "password": "string (required)"
}
```

**レスポンス (200 OK):**
```json
{
  "user": {
    "id": 1,
    "user_code": "user001",
    "name": "山田太郎",
    "email": "yamada@example.com",
    "role": "スタッフ"
  },
  "token": "1|abcdef123456...",
  "expires_at": "2025-07-17T15:30:00Z"
}
```

**エラーレスポンス (401 Unauthorized):**
```json
{
  "message": "認証に失敗しました",
  "errors": {
    "user_code": ["ユーザーコードまたはパスワードが正しくありません"]
  }
}
```

#### POST /api/logout
現在のセッションを終了し、トークンを無効化します。

**ヘッダー:**
```
Authorization: Bearer {token}
```

**レスポンス (204 No Content):**
```
(空のレスポンス)
```

#### POST /api/auth/refresh
トークンを更新します。

**ヘッダー:**
```
Authorization: Bearer {token}
```

**レスポンス (200 OK):**
```json
{
  "token": "2|newtoken123456...",
  "expires_at": "2025-07-17T18:30:00Z"
}
```

### 認証ユーザー情報取得

#### GET /api/user
現在認証されているユーザーの情報を取得します。

**ヘッダー:**
```
Authorization: Bearer {token}
```

**レスポンス (200 OK):**
```json
{
  "id": 1,
  "user_code": "user001",
  "name": "山田太郎",
  "email": "yamada@example.com",
  "role": "スタッフ",
  "phone_number": "03-1234-5678",
  "mobile_phone_number": "090-1234-5678",
  "avatar_photo": "/storage/avatars/user001.jpg",
  "created_at": "2025-07-01T10:00:00Z",
  "updated_at": "2025-07-15T14:30:00Z"
}
```

## スタッフ管理API

### スタッフ一覧取得

#### GET /api/staff
スタッフ一覧を取得します。検索、フィルタリング、ページネーション機能付き。

**クエリパラメータ:**
- `page` (optional): ページ番号 (default: 1)
- `per_page` (optional): 1ページあたりの件数 (default: 20, max: 100)
- `search` (optional): 検索キーワード (氏名、メール、電話番号で検索)
- `role` (optional): 役職でフィルタリング
- `sort` (optional): ソート項目 (name, email, created_at)
- `order` (optional): ソート順 (asc, desc)

**レスポンス (200 OK):**
```json
{
  "data": [
    {
      "id": 1,
      "user_code": "user001",
      "name": "山田太郎",
      "email": "yamada@example.com",
      "role": "スタッフ",
      "phone_number": "03-1234-5678",
      "mobile_phone_number": "090-1234-5678",
      "avatar_photo": "/storage/avatars/user001.jpg",
      "created_at": "2025-07-01T10:00:00Z",
      "updated_at": "2025-07-15T14:30:00Z"
    }
  ],
  "links": {
    "first": "/api/staff?page=1",
    "last": "/api/staff?page=5",
    "prev": null,
    "next": "/api/staff?page=2"
  },
  "meta": {
    "current_page": 1,
    "from": 1,
    "last_page": 5,
    "per_page": 20,
    "to": 20,
    "total": 100
  }
}
```

### スタッフ詳細取得

#### GET /api/staff/{id}
指定されたスタッフの詳細情報を取得します。

**レスポンス (200 OK):**
```json
{
  "id": 1,
  "user_code": "user001",
  "name": "山田太郎",
  "email": "yamada@example.com",
  "role": "スタッフ",
  "phone_number": "03-1234-5678",
  "mobile_phone_number": "090-1234-5678",
  "avatar_photo": "/storage/avatars/user001.jpg",
  "created_at": "2025-07-01T10:00:00Z",
  "updated_at": "2025-07-15T14:30:00Z"
}
```

**エラーレスポンス (404 Not Found):**
```json
{
  "message": "スタッフが見つかりません"
}
```

### スタッフ新規登録

#### POST /api/staff
新しいスタッフを登録します。

**リクエスト:**
```json
{
  "user_code": "string (required, unique, max:20)",
  "name": "string (required, max:50)",
  "email": "string (required, email, unique)",
  "password": "string (required, min:8)",
  "role": "string (required, in:全権管理者,一般管理者,スタッフ)",
  "phone_number": "string (optional, max:15)",
  "mobile_phone_number": "string (optional, max:15)",
  "avatar_photo": "file (optional, image, max:2MB)"
}
```

**レスポンス (201 Created):**
```json
{
  "id": 2,
  "user_code": "user002",
  "name": "田中花子",
  "email": "tanaka@example.com",
  "role": "スタッフ",
  "phone_number": "03-5678-1234",
  "mobile_phone_number": "090-5678-1234",
  "avatar_photo": "/storage/avatars/user002.jpg",
  "created_at": "2025-07-17T10:00:00Z",
  "updated_at": "2025-07-17T10:00:00Z"
}
```

**エラーレスポンス (422 Unprocessable Entity):**
```json
{
  "message": "バリデーションエラー",
  "errors": {
    "user_code": ["このユーザーコードは既に使用されています"],
    "email": ["このメールアドレスは既に登録されています"],
    "password": ["パスワードは英数字を含む8文字以上で入力してください"]
  }
}
```

### スタッフ情報更新

#### PUT /api/staff/{id}
指定されたスタッフの情報を更新します。

**リクエスト:**
```json
{
  "name": "string (optional, max:50)",
  "email": "string (optional, email, unique)",
  "role": "string (optional, in:全権管理者,一般管理者,スタッフ)",
  "phone_number": "string (optional, max:15)",
  "mobile_phone_number": "string (optional, max:15)",
  "avatar_photo": "file (optional, image, max:2MB)"
}
```

**レスポンス (200 OK):**
```json
{
  "id": 1,
  "user_code": "user001",
  "name": "山田太郎",
  "email": "yamada@example.com",
  "role": "一般管理者",
  "phone_number": "03-1234-5678",
  "mobile_phone_number": "090-1234-5678",
  "avatar_photo": "/storage/avatars/user001.jpg",
  "created_at": "2025-07-01T10:00:00Z",
  "updated_at": "2025-07-17T11:00:00Z"
}
```

### スタッフ削除

#### DELETE /api/staff/{id}
指定されたスタッフを削除します。

**レスポンス (204 No Content):**
```
(空のレスポンス)
```

**エラーレスポンス (403 Forbidden):**
```json
{
  "message": "削除権限がありません"
}
```

## メール送信API

### メール一括送信

#### POST /api/mail/send
選択されたスタッフに対してメールを一括送信します。

**リクエスト:**
```json
{
  "subject": "string (required, max:255)",
  "body": "string (required)",
  "recipients": "array (required, min:1)",
  "recipients.*": "integer (exists:users,id)",
  "attachments": "array (optional)",
  "attachments.*": "file (max:10MB)"
}
```

**レスポンス (200 OK):**
```json
{
  "message": "メール送信が完了しました",
  "broadcast_id": 1,
  "sent_count": 5,
  "failed_count": 0,
  "recipients": [
    {
      "id": 1,
      "name": "山田太郎",
      "email": "yamada@example.com",
      "status": "sent"
    }
  ]
}
```

### メール送信履歴

#### GET /api/mail/history
メール送信履歴を取得します。

**クエリパラメータ:**
- `page` (optional): ページ番号
- `per_page` (optional): 1ページあたりの件数

**レスポンス (200 OK):**
```json
{
  "data": [
    {
      "id": 1,
      "subject": "重要なお知らせ",
      "body": "本日は重要なお知らせがあります...",
      "sent_at": "2025-07-17T10:00:00Z",
      "status": "sent",
      "recipients_count": 5,
      "sender": {
        "id": 1,
        "name": "管理者",
        "user_code": "admin001"
      }
    }
  ],
  "meta": {
    "current_page": 1,
    "last_page": 3,
    "per_page": 10,
    "total": 25
  }
}
```

## セキュリティAPI

### セキュリティ違反報告

#### POST /api/security/violation
セキュリティ違反を報告します。

**リクエスト:**
```json
{
  "type": "string (required)",
  "url": "string (required)",
  "details": "string (optional)"
}
```

**レスポンス (200 OK):**
```json
{
  "message": "セキュリティ違反が記録されました",
  "redirect": "/security-error"
}
```

## 共通エラーレスポンス

### 400 Bad Request
```json
{
  "message": "リクエストが正しくありません",
  "errors": {
    "field": ["具体的なエラーメッセージ"]
  }
}
```

### 401 Unauthorized
```json
{
  "message": "認証が必要です"
}
```

### 403 Forbidden
```json
{
  "message": "この操作を実行する権限がありません"
}
```

### 404 Not Found
```json
{
  "message": "リソースが見つかりません"
}
```

### 422 Unprocessable Entity
```json
{
  "message": "バリデーションエラー",
  "errors": {
    "field": ["具体的なエラーメッセージ"]
  }
}
```

### 429 Too Many Requests
```json
{
  "message": "リクエスト数が上限に達しています。しばらくしてから再試行してください",
  "retry_after": 60
}
```

### 500 Internal Server Error
```json
{
  "message": "内部サーバーエラーが発生しました"
}
```

## レート制限

- **認証API**: 10回/分
- **一般API**: 100回/分
- **メール送信**: 5回/分

## ファイルアップロード

### 対応フォーマット
- **画像**: JPEG, PNG, GIF, WebP
- **添付ファイル**: PDF, DOC, DOCX, XLS, XLSX

### サイズ制限
- **アバター画像**: 最大2MB
- **メール添付**: 最大10MB

### セキュリティ対策
- ファイル形式検証
- ウイルススキャン
- 安全なファイル名生成
- 適切なMIMEタイプ設定

## APIバージョニング

現在のAPIバージョン: v1
- 将来的な変更時はバージョニングを実装
- 互換性のない変更は新バージョンで提供
- 旧バージョンは6ヶ月間サポート

## 開発・テスト環境

### 開発環境
- URL: `http://localhost:8000/api`
- デバッグモード: 有効
- 詳細エラー情報: 表示

### 本番環境
- URL: `https://your-app.fly.dev/api`
- デバッグモード: 無効
- 詳細エラー情報: 非表示
