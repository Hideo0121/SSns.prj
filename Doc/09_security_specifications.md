# セキュリティ仕様書

このドキュメントでは、スタッフ管理システムのセキュリティ仕様について詳細に説明します。

## セキュリティ概要

### セキュリティ目標
- **機密性**: 認証されたユーザーのみがデータにアクセス可能
- **完全性**: データの改ざん防止と検証
- **可用性**: 正当なユーザーのアクセスを保証
- **追跡可能性**: 全ての操作の監査ログ記録

### 脅威モデル
- **不正アクセス**: 認証されていないユーザーによるシステムアクセス
- **権限昇格**: 一般ユーザーが管理者権限を取得
- **データ漏洩**: 機密情報の不正な流出
- **セッションハイジャック**: 他人のセッションを乗っ取り
- **XSS攻撃**: 悪意のあるスクリプトの実行
- **CSRF攻撃**: 偽装されたリクエストによる不正操作

## 認証・認可

### 認証システム
- **方式**: Laravel Sanctum (Personal Access Token)
- **パスワード**: bcryptでハッシュ化 (cost: 12)
- **認証情報**: user_code + password
- **トークン有効期限**: 3時間
- **自動延長**: 残り30分で自動更新

### 認証強化策
```php
// パスワード要件
- 最小8文字以上
- 英数字を含む
- 特殊文字推奨
- 辞書攻撃対策

// アカウントロック
- 5回連続失敗でアカウントロック
- ロック時間: 15分
- 段階的ロック時間増加
```

### 認可システム
```php
// 権限レベル
Role::SUPER_ADMIN    // 全権管理者
Role::ADMIN          // 一般管理者  
Role::STAFF          // スタッフ

// 権限マトリックス
Action::CREATE_USER   -> [SUPER_ADMIN, ADMIN]
Action::DELETE_USER   -> [SUPER_ADMIN]
Action::SEND_MAIL     -> [SUPER_ADMIN, ADMIN]
Action::VIEW_LOGS     -> [SUPER_ADMIN]
```

## セッション管理

### セッション仕様
- **セッションID**: 128bit暗号化
- **セッション有効期限**: 3時間
- **セッションストレージ**: データベース
- **セッション再生成**: ログイン時に必須

### セッション保護策
```javascript
// クライアントサイド保護
- sessionStorage使用 (localStorage×)
- タブクローズ時の自動削除
- 非アクティブ時のタイムアウト
- 複数タブ・ウィンドウ制御

// サーバーサイド保護
- セッションフィクセーション対策
- セッション再生成の実装
- IPアドレス変更検知
- User-Agent変更検知
```

## 入力値検証

### バリデーション階層
1. **クライアントサイド**: 即座のフィードバック
2. **サーバーサイド**: 必須の検証
3. **データベース**: 制約による最終防御

### 入力値サニタイゼーション
```php
// HTMLエスケープ
htmlspecialchars($input, ENT_QUOTES, 'UTF-8');

// SQLインジェクション対策
- Eloquent ORM使用
- プリペアドステートメント
- パラメータバインディング

// XSS対策
- CSRFトークン検証
- Content Security Policy
- 出力時エスケープ
```

### ファイルアップロード検証
```php
// ファイル検証項目
- 拡張子チェック
- MIMEタイプ検証
- ファイルサイズ制限
- ウイルススキャン
- 実行可能ファイル除外
```

## 暗号化

### データ暗号化
```php
// 保存時暗号化
- パスワード: bcrypt (cost: 12)
- セッションデータ: AES-256-CBC
- 機密データ: Laravel Encryption
- データベース: MySQL TDE (本番環境)

// 通信暗号化
- HTTPS/TLS 1.3
- HSTS Header
- Certificate Pinning
```

### 暗号化実装例
```php
// パスワードハッシュ化
$hashedPassword = bcrypt($password);

// データ暗号化
$encryptedData = encrypt($sensitiveData);

// トークン生成
$token = Str::random(60);
```

## CSRF対策

### CSRF保護実装
```php
// Laravel標準CSRF保護
'csrf' => [
    'verify' => true,
    'token_lifetime' => 3600, // 1時間
    'regenerate' => true,
];

// SameSite Cookie
'same_site' => 'strict',
'secure' => true,
'http_only' => true,
```

### CSRFトークン検証
```javascript
// APIリクエストヘッダー
headers: {
    'X-CSRF-TOKEN': document.querySelector('meta[name="csrf-token"]').getAttribute('content'),
    'X-Requested-With': 'XMLHttpRequest'
}
```

## XSS対策

### Content Security Policy
```php
// CSP設定
"default-src 'self'",
"script-src 'self' 'unsafe-inline'",
"style-src 'self' 'unsafe-inline' https://fonts.googleapis.com",
"font-src 'self' https://fonts.gstatic.com",
"img-src 'self' data:",
"connect-src 'self'",
"frame-ancestors 'none'",
"base-uri 'self'",
"form-action 'self'",
"upgrade-insecure-requests"
```

### XSS防御策
```php
// 出力エスケープ
{{{ $variable }}}  // 自動エスケープ
{{ $variable }}    // 手動エスケープ

// HTMLパージ
$clean = strip_tags($input);
$clean = htmlspecialchars($input, ENT_QUOTES, 'UTF-8');
```

## セキュリティヘッダー

### 必須セキュリティヘッダー
```php
// セキュリティヘッダー設定
'security.headers' => [
    'X-Frame-Options' => 'DENY',
    'X-Content-Type-Options' => 'nosniff',
    'X-XSS-Protection' => '1; mode=block',
    'Strict-Transport-Security' => 'max-age=31536000; includeSubDomains',
    'Referrer-Policy' => 'strict-origin-when-cross-origin',
    'Permissions-Policy' => 'camera=(), microphone=(), geolocation=()',
];
```

## レート制限

### API制限設定
```php
// レート制限設定
'api_rate_limit' => [
    'login' => '10,1',      // 10回/分
    'general' => '100,1',   // 100回/分
    'mail' => '5,1',        // 5回/分
    'upload' => '20,1',     // 20回/分
];

// 制限超過時の対応
- 429 Too Many Requests
- Retry-After ヘッダー
- 段階的制限強化
```

## 監査ログ

### ログ記録対象
```php
// 認証関連
- ログイン成功/失敗
- ログアウト
- パスワード変更
- 権限変更

// データ操作
- CRUD操作
- ファイルアップロード
- メール送信
- データエクスポート

// セキュリティ関連
- セキュリティ違反
- 異常なアクセス
- 権限エラー
- システムエラー
```

### ログ形式
```json
{
  "timestamp": "2025-07-17T10:00:00Z",
  "level": "INFO",
  "event": "user.login",
  "user_id": 1,
  "user_code": "user001",
  "ip_address": "192.168.1.100",
  "user_agent": "Mozilla/5.0...",
  "details": {
    "action": "login_success",
    "session_id": "abc123..."
  }
}
```

## 脆弱性対策

### 定期的なセキュリティ対策
```bash
# 依存関係の脆弱性チェック
composer audit
npm audit

# セキュリティパッチ適用
composer update
npm update

# 定期的なペネトレーションテスト
# 脆弱性スキャン
# コードレビュー
```

### セキュリティテスト項目
- **認証バイパス**: 認証機能の迂回テスト
- **権限昇格**: 不正な権限取得テスト
- **SQLインジェクション**: データベース攻撃テスト
- **XSS**: クロスサイトスクリプティングテスト
- **CSRF**: クロスサイトリクエストフォージェリテスト
- **セッション管理**: セッション関連の脆弱性テスト

## インシデント対応

### セキュリティインシデント分類
```php
// 重要度レベル
Level::CRITICAL  // システム停止、データ漏洩
Level::HIGH      // 不正アクセス、権限昇格
Level::MEDIUM    // 異常なアクセス、システムエラー
Level::LOW       // 軽微な警告、設定変更
```

### 対応手順
1. **検知**: 自動監視・手動発見
2. **分析**: 影響範囲・原因調査
3. **封じ込め**: 攻撃の停止・拡散防止
4. **復旧**: システム・データの復旧
5. **学習**: 再発防止策の実装

## データ保護

### 個人情報保護
```php
// 個人情報の分類
- 氏名、メールアドレス
- 電話番号
- 顔写真
- IPアドレス
- アクセスログ

// 保護策
- 暗号化保存
- アクセス制御
- 保存期間制限
- 削除機能
```

### データ匿名化
```php
// 個人情報の匿名化
$anonymized = [
    'name' => '***',
    'email' => hash('sha256', $email),
    'phone' => substr($phone, 0, 3) . '****',
];
```

## 本番環境セキュリティ

### 本番環境での追加セキュリティ
```php
// 本番環境設定
APP_DEBUG=false
APP_ENV=production

// データベース接続
- 専用ユーザー作成
- 最小権限の原則
- 接続暗号化

// サーバー設定
- ファイアウォール設定
- 不要サービス停止
- OS・ミドルウェア更新
```

### 監視・アラート
```php
// 監視対象
- 異常なアクセス
- エラー率の増加
- レスポンス時間の悪化
- リソース使用量

// アラート設定
- 即座通知: 重要なセキュリティイベント
- 日次レポート: アクセス統計
- 週次レポート: セキュリティ状況
```

## セキュリティチェックリスト

### 開発時チェック
- [ ] 入力値検証の実装
- [ ] 出力エスケープの実装
- [ ] 認証・認可の実装
- [ ] CSRFトークンの検証
- [ ] セキュリティヘッダーの設定
- [ ] ログ機能の実装

### デプロイ時チェック
- [ ] 本番環境設定の確認
- [ ] HTTPS/TLS設定
- [ ] データベース接続設定
- [ ] 監視・アラート設定
- [ ] バックアップ設定
- [ ] セキュリティテストの実行

### 運用時チェック
- [ ] 定期的なセキュリティ更新
- [ ] ログ監視
- [ ] 脆弱性スキャン
- [ ] ペネトレーションテスト
- [ ] インシデント対応計画の更新
- [ ] セキュリティ教育の実施
