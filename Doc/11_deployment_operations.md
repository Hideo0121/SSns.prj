# デプロイ・運用手順書 (Fly.io + Supabase)

このドキュメントでは、Fly.io + Supabase を使用したスタッフ管理システムのデプロイ・運用手順について詳細に説明します。

## システム構成

### 本番環境アーキテクチャ
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   Backend       │    │   Database      │
│   (React)       │◄──►│   (Laravel)     │◄──►│   (Supabase)    │
│   Fly.io        │    │   Fly.io        │    │   PostgreSQL    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### 使用サービス
- **Fly.io**: アプリケーションホスティング
- **Supabase**: PostgreSQL データベース + 認証
- **GitHub**: ソースコード管理
- **GitHub Actions**: CI/CDパイプライン

## 事前準備

### 1. 必要なツールのインストール

#### Fly.io CLI
```bash
# macOS
brew install flyctl

# Linux
curl -L https://fly.io/install.sh | sh

# Windows
powershell -Command "iwr https://fly.io/install.ps1 -useb | iex"
```

#### Supabase CLI
```bash
# npm経由
npm install -g supabase

# brew経由 (macOS)
brew install supabase/tap/supabase
```

### 2. アカウント作成・認証

#### Fly.io
```bash
# アカウント作成・ログイン
flyctl auth signup
flyctl auth login

# 組織確認
flyctl orgs list
```

#### Supabase
```bash
# ログイン
supabase login

# プロジェクト作成
supabase projects create staff-management
```

## データベース設定 (Supabase)

### 1. Supabaseプロジェクトの設定

#### プロジェクト作成
```bash
# Supabaseプロジェクト作成
supabase projects create staff-management-system

# 設定情報の確認
supabase projects list
```

#### 接続情報の取得
```bash
# データベース接続情報
supabase db show --db-url
```

### 2. データベースマイグレーション

#### スキーマ作成
```sql
-- users テーブル
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    user_code VARCHAR(20) UNIQUE NOT NULL,
    name VARCHAR(50) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    role VARCHAR(50) NOT NULL CHECK (role IN ('全権管理者', '一般管理者', 'スタッフ')),
    phone_number VARCHAR(15),
    mobile_phone_number VARCHAR(15),
    avatar_photo TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- インデックス作成
CREATE INDEX idx_users_user_code ON users(user_code);
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);
```

#### Row Level Security (RLS) 設定
```sql
-- RLS有効化
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- ポリシー作成
CREATE POLICY "Users can view their own profile" ON users
    FOR SELECT USING (auth.uid() = id);

CREATE POLICY "Admins can view all users" ON users
    FOR SELECT USING (
        EXISTS (
            SELECT 1 FROM users 
            WHERE id = auth.uid() 
            AND role IN ('全権管理者', '一般管理者')
        )
    );
```

### 3. Supabase設定ファイル

#### supabase/config.toml
```toml
[project]
ref = "your-project-ref"
org_id = "your-org-id"

[database]
port = 54322
shadow_port = 54320
```

## アプリケーションのデプロイ準備

### 1. Laravel設定

#### fly.toml作成
```toml
app = "staff-management-system"
primary_region = "nrt"

[build]
  image = "registry.fly.io/staff-management-system:latest"

[env]
  APP_ENV = "production"
  APP_DEBUG = "false"
  LOG_CHANNEL = "stderr"
  LOG_LEVEL = "info"
  DB_CONNECTION = "pgsql"

[processes]
  app = "php artisan serve --host=0.0.0.0 --port=8000"
  worker = "php artisan queue:work"

[[services]]
  internal_port = 8000
  protocol = "tcp"
  
  [services.concurrency]
    hard_limit = 25
    soft_limit = 20
    type = "connections"

  [[services.ports]]
    force_https = true
    handlers = ["http"]
    port = 80

  [[services.ports]]
    handlers = ["tls", "http"]
    port = 443

  [services.http_checks]
    interval = 10000
    grace_period = "5s"
    method = "get"
    path = "/health"
    protocol = "http"
    timeout = 2000
    tls_skip_verify = false

[metrics]
  port = 9091
  path = "/metrics"
```

#### Dockerfile作成
```dockerfile
FROM php:8.2-fpm

# Install system dependencies
RUN apt-get update && apt-get install -y \
    git \
    curl \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    libpq-dev \
    zip \
    unzip \
    nodejs \
    npm

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Install PHP extensions
RUN docker-php-ext-install pdo_pgsql pgsql mbstring exif pcntl bcmath gd

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Set working directory
WORKDIR /var/www

# Copy application
COPY . .

# Install dependencies
RUN composer install --no-dev --optimize-autoloader
RUN npm ci && npm run build

# Set permissions
RUN chown -R www-data:www-data /var/www/storage /var/www/bootstrap/cache

# Expose port
EXPOSE 8000

# Start application
CMD ["php", "artisan", "serve", "--host=0.0.0.0", "--port=8000"]
```

### 2. 環境変数設定

#### Fly.io Secrets
```bash
# アプリケーション設定
flyctl secrets set APP_KEY="base64:$(openssl rand -base64 32)"
flyctl secrets set APP_URL="https://staff-management-system.fly.dev"

# データベース設定
flyctl secrets set DB_HOST="your-supabase-host"
flyctl secrets set DB_PORT="5432"
flyctl secrets set DB_DATABASE="postgres"
flyctl secrets set DB_USERNAME="postgres"
flyctl secrets set DB_PASSWORD="your-supabase-password"

# Supabase設定
flyctl secrets set SUPABASE_URL="https://your-project.supabase.co"
flyctl secrets set SUPABASE_ANON_KEY="your-anon-key"
flyctl secrets set SUPABASE_SERVICE_ROLE_KEY="your-service-role-key"

# メール設定
flyctl secrets set MAIL_MAILER="smtp"
flyctl secrets set MAIL_HOST="your-smtp-host"
flyctl secrets set MAIL_PORT="587"
flyctl secrets set MAIL_USERNAME="your-email"
flyctl secrets set MAIL_PASSWORD="your-password"

# セキュリティ設定
flyctl secrets set SANCTUM_STATEFUL_DOMAINS="staff-management-system.fly.dev"
flyctl secrets set SESSION_DOMAIN="staff-management-system.fly.dev"
```

## CI/CD パイプライン

### 1. GitHub Actions設定

#### .github/workflows/deploy.yml
```yaml
name: Deploy to Fly.io

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:13
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: testing
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

    steps:
    - uses: actions/checkout@v3
    
    - name: Setup PHP
      uses: shivammathur/setup-php@v2
      with:
        php-version: '8.2'
        extensions: mbstring, xml, ctype, iconv, intl, pdo_pgsql
    
    - name: Install dependencies
      run: composer install --prefer-dist --no-progress
    
    - name: Create .env.testing
      run: |
        cp .env.example .env.testing
        php artisan key:generate --env=testing
    
    - name: Run tests
      run: php artisan test
      env:
        DB_CONNECTION: pgsql
        DB_HOST: localhost
        DB_PORT: 5432
        DB_DATABASE: testing
        DB_USERNAME: postgres
        DB_PASSWORD: postgres

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Fly.io
      uses: superfly/flyctl-actions/setup-flyctl@master
    
    - name: Deploy to Fly.io
      run: flyctl deploy --remote-only
      env:
        FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}
```

### 2. コードレビュー手順

#### プルリクエスト作成
```bash
# feature ブランチで作業
git checkout -b feature/new-feature
git add .
git commit -m "Add new feature"
git push origin feature/new-feature

# GitHub でプルリクエスト作成
# レビュー完了後、main ブランチにマージ
```

#### コードレビューチェックリスト
- [ ] コードの品質とベストプラクティス
- [ ] セキュリティ要件の確認
- [ ] テストケースの実装
- [ ] ドキュメント更新
- [ ] 破壊的変更の確認

## デプロイ手順

### 1. 初回デプロイ

#### Fly.io アプリケーション作成
```bash
# アプリケーション作成
flyctl apps create staff-management-system

# 設定確認
flyctl apps list
```

#### データベースマイグレーション
```bash
# Supabase マイグレーション実行
supabase db push

# Laravel マイグレーション実行
flyctl ssh console
php artisan migrate --force
```

#### 初回デプロイ実行
```bash
# Docker イメージビルド・デプロイ
flyctl deploy

# デプロイ状況確認
flyctl status
flyctl logs
```

### 2. 定期デプロイ

#### 自動デプロイ
- main ブランチへのマージで自動実行
- GitHub Actions によるテスト・デプロイ
- 失敗時の自動ロールバック

#### 手動デプロイ
```bash
# 手動デプロイ実行
flyctl deploy --remote-only

# 特定バージョンのデプロイ
flyctl deploy --image registry.fly.io/staff-management-system:v1.0.0
```

## 運用監視

### 1. ログ監視

#### アプリケーションログ
```bash
# リアルタイムログ
flyctl logs

# 特定時間のログ
flyctl logs --since=1h

# エラーログのみ
flyctl logs | grep ERROR
```

#### データベースログ
```bash
# Supabase ログ確認
supabase logs --type=database

# 設定情報確認
supabase status
```

### 2. パフォーマンス監視

#### Fly.io メトリクス
```bash
# CPU・メモリ使用率
flyctl metrics

# 接続状況
flyctl status --watch

# スケーリング状況
flyctl scale show
```

#### データベース監視
```bash
# 接続数確認
supabase db logs --type=connections

# クエリ実行状況
supabase db logs --type=queries
```

### 3. アラート設定

#### Fly.io アラート
```bash
# メール通知設定
flyctl monitor --email your-email@example.com

# Slack通知設定
flyctl monitor --slack-webhook YOUR_WEBHOOK_URL
```

## バックアップ・復旧

### 1. データベースバックアップ

#### 定期バックアップ設定
```bash
# Supabase 自動バックアップ設定
supabase db backup --schedule daily

# 手動バックアップ
supabase db backup --output backup.sql
```

#### バックアップ確認
```bash
# バックアップ一覧
supabase db backups list

# バックアップ詳細
supabase db backups get backup-id
```

### 2. アプリケーションバックアップ

#### 設定ファイルバックアップ
```bash
# 環境変数バックアップ
flyctl secrets list > secrets_backup.txt

# 設定ファイルバックアップ
cp fly.toml fly.toml.backup
```

### 3. 復旧手順

#### データベース復旧
```bash
# バックアップから復旧
supabase db restore backup-id

# 特定時点への復旧
supabase db restore --point-in-time "2025-07-17T10:00:00Z"
```

#### アプリケーション復旧
```bash
# 前バージョンへのロールバック
flyctl releases list
flyctl releases rollback v1

# 緊急時の強制復旧
flyctl deploy --image registry.fly.io/staff-management-system:stable
```

## セキュリティ対策

### 1. 本番環境セキュリティ

#### SSL/TLS設定
```bash
# 自動SSL証明書
flyctl certs create staff-management-system.fly.dev

# カスタムドメイン
flyctl certs create your-domain.com
```

#### ファイアウォール設定
```bash
# IPアドレス制限
flyctl ips allocate-v4 --region nrt
flyctl ips allocate-v6 --region nrt
```

### 2. データベースセキュリティ

#### Supabase セキュリティ設定
```sql
-- IP制限
ALTER SYSTEM SET listen_addresses = 'specific-ip-range';

-- 接続暗号化
ALTER SYSTEM SET ssl = on;
ALTER SYSTEM SET ssl_cert_file = 'server.crt';
ALTER SYSTEM SET ssl_key_file = 'server.key';
```

### 3. 監査ログ

#### セキュリティログ監視
```bash
# 認証ログ
flyctl logs | grep "auth"

# セキュリティ違反ログ
flyctl logs | grep "security_violation"
```

## 運用チェックリスト

### 日次チェック
- [ ] システム稼働状況確認
- [ ] エラーログ確認
- [ ] パフォーマンス指標確認
- [ ] データベース接続状況確認

### 週次チェック
- [ ] システムリソース使用状況
- [ ] セキュリティログ分析
- [ ] バックアップ実行状況確認
- [ ] 依存関係更新確認

### 月次チェック
- [ ] セキュリティパッチ適用
- [ ] パフォーマンスチューニング
- [ ] データベース最適化
- [ ] 運用レポート作成

## トラブルシューティング

### 1. 一般的な問題

#### デプロイ失敗
```bash
# ログ確認
flyctl logs --app staff-management-system

# ヘルスチェック確認
flyctl status --app staff-management-system

# 設定確認
flyctl config show
```

#### データベース接続エラー
```bash
# 接続テスト
flyctl ssh console
php artisan tinker
DB::connection()->getPdo()

# Supabase 接続確認
supabase status
```

### 2. 緊急時対応

#### システム停止時
1. 問題の特定と分析
2. 緊急パッチの適用
3. サービス復旧
4. 影響範囲の確認

#### データ破損時
1. バックアップからの復旧
2. データ整合性の確認
3. 原因分析と対策
4. 再発防止策の実装

## メンテナンス

### 1. 定期メンテナンス

#### アプリケーション更新
```bash
# 依存関係更新
composer update
npm update

# セキュリティパッチ適用
composer audit
npm audit fix
```

#### データベースメンテナンス
```bash
# インデックス再構築
supabase db vacuum

# 統計情報更新
supabase db analyze
```

### 2. スケーリング

#### 水平スケーリング
```bash
# インスタンス数増加
flyctl scale count 3

# 地域展開
flyctl scale regions add sin
```

#### 垂直スケーリング
```bash
# CPU・メモリ増強
flyctl scale vm shared-cpu-2x

# 専用CPU
flyctl scale vm dedicated-cpu-1x
```

## 継続的改善

### 1. パフォーマンス改善
- レスポンス時間の最適化
- データベースクエリの改善
- キャッシュ戦略の見直し

### 2. セキュリティ強化
- 脆弱性スキャンの定期実行
- セキュリティポリシーの更新
- 監査ログの充実

### 3. 運用効率化
- 自動化スクリプトの拡充
- 監視アラートの最適化
- ドキュメント更新
