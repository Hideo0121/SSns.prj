# 技術スタックとセットアップ方法

## 技術スタック

* **バックエンド**: **PHP + Laravel**
* **フロントエンド**: **Tailwind CSS** と **React** を利用し、動的でインタラクティブなページを構築します。
* **UIコンポーネント**: テキスト、ラベル、ボタンなどのUIパーツには **Material-UI** を利用します。
* **データベース**: **MySQL** を使用します。
* **LINE Messaging API**: LINE公式アカウントとの連携に必須。
* **メール送信サービス**: 例: SendGrid, Postmark など。

---

## セットアップ方法 (開発環境)

1.  **リポジトリのクローン:**
    ```bash
    git clone [リポジトリのURL]
    cd [プロジェクトディレクトリ名]
    ```
2.  **Composer依存関係のインストール:**
    ```bash
    composer install
    ```
3.  **JavaScript依存関係のインストール (フロントエンドにReactを使用するため):**
    ```bash
    npm install # または yarn install
    ```
4.  **.envファイルの作成と設定:**
    ```bash
    cp .env.example .env
    ```
    `.env`ファイルを開き、データベース接続情報と**LINE Developers**から取得した情報を設定してください。
    ```env
    DB_CONNECTION=mysql
    DB_HOST=127.0.0.1
    DB_PORT=3306
    DB_DATABASE=your_database_name
    DB_USERNAME=your_username
    DB_PASSWORD=your_password

    # LINE Messaging API Credentials
    LINE_CHANNEL_ACCESS_TOKEN=your_line_channel_access_token
    LINE_CHANNEL_SECRET=your_line_channel_secret

    # Mail Server Credentials (if applicable)
    MAIL_MAILER=smtp
    MAIL_HOST=your_mail_host
    MAIL_PORT=587
    MAIL_USERNAME=your_mail_username
    MAIL_PASSWORD=your_mail_password
    MAIL_ENCRYPTION=tls
    MAIL_FROM_ADDRESS=your_from_email@example.com
    MAIL_FROM_NAME="${APP_NAME}"
    ```
5.  **アプリケーションキーの生成:**
    ```bash
    php artisan key:generate
    ```
6.  **データベースマイグレーションの実行:**
    ```bash
    php artisan migrate
    ```
7.  **フロントエンドのビルド (開発用):**
    ```bash
    npm run dev # または yarn dev
    ```
8.  **開発サーバーの起動:**
    ```bash
    php artisan serve
    ```
    ブラウザで `http://127.0.0.1:8000` にアクセスして、アプリケーションを確認してください。
