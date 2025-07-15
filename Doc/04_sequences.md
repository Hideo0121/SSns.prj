# シーケンス

このセクションでは、主要な機能におけるシステムとユーザー、および外部システム（LINE APIなど）とのインタラクションのシーケンス図を記述します。

### 登場人物（アクターとシステムコンポーネント）

* **管理者** (AdminUser): システムを操作する全権管理者または一般管理者。
* **ブラウザ** (Browser): Webブラウザ。
* **Laravel Application** (WebServer): PHP Laravelで構築されたWebアプリケーションサーバー。
* **データベース** (Database): スタッフ情報やシステムデータを管理するデータベース。
* **LINE Messaging API** (LineAPI): LINE公式アカウントと連携するためのAPI。
* **メール送信サービス** (MailService): システムからメールを送信するための外部サービス（例: SendGrid, Postmarkなど）。

### ① 認証 ⇒ スタッフ検索・一覧 ⇒ スタッフ詳細画面でのメンテナンス＆登録

```mermaid
sequenceDiagram
    participant AdminUser as 管理者
    participant Browser
    participant WebServer as Laravel Application
    participant Database

    AdminUser->>Browser: 認証画面にアクセス
    Browser->>WebServer: GET /login
    WebServer->>Browser: 認証画面HTMLを返す
    AdminUser->>Browser: ID/パスワード入力、ログインクリック
    Browser->>WebServer: POST /login (認証情報)
    WebServer->>Database: 認証情報の照合を要求
    Database-->>WebServer: 認証結果を返す
    alt 認証成功
        WebServer->>Browser: スタッフ検索・一覧画面HTMLを返す
        AdminUser->>Browser: スタッフを検索・フィルタリング
        Browser->>WebServer: 検索条件を送信、スタッフリスト取得要求
        WebServer->>Database: 条件に合致するスタッフリスト取得要求
        Database-->>WebServer: スタッフリストを返す
        WebServer->>Browser: スタッフリストを返す
        Browser->>AdminUser: スタッフリストを表示
        AdminUser->>Browser: 特定のスタッフを選択 (チェックボックス)
        alt 複数選択時
            AdminUser->>Browser: (詳細表示ボタンはクリックできない)
        else 単数選択時
            AdminUser->>Browser: 詳細表示ボタンをクリック
            Browser->>WebServer: 選択スタッフの詳細情報表示要求
            WebServer->>Database: スタッフ詳細情報の取得要求
            Database-->>WebServer: スタッフ詳細情報を返す
            WebServer->>Browser: スタッフ詳細画面HTMLを返す
            AdminUser->>Browser: スタッフ情報を編集、更新クリック
            Browser->>WebServer: POST /staff/{id} (更新データ)
            WebServer->>WebServer: 入力値のバリデーション
            alt バリデーション成功
                WebServer->>Database: スタッフ情報の更新を要求
                Database-->>WebServer: 更新完了を通知
                WebServer->>WebServer: メンテナンスログを記録
                WebServer->>Database: メンテナンスログ保存
                Database-->>WebServer: ログ保存完了
                WebServer->>Browser: 更新完了メッセージ、スタッフ一覧へリダイレクト
            else バリデーション失敗
                WebServer->>Browser: エラーメッセージと共に詳細画面を再表示
            end
        end
    else 認証失敗
        WebServer->>Browser: エラーメッセージと共に認証画面を再表示
    end
```

### ② 認証 ⇒ スタッフ検索・一覧 ⇒ LINE送信画面から本文入力し送信

```mermaid
sequenceDiagram
    participant AdminUser as 管理者
    participant Browser
    participant WebServer as Laravel Application
    participant Database
    participant LineAPI as LINE Messaging API

    AdminUser->>Browser: 認証画面にアクセス (認証フローは省略)
    Browser->>WebServer: ...
    WebServer->>Database: ...
    Database-->>WebServer: ...
    WebServer->>Browser: スタッフ検索・一覧画面HTMLを返す (認証成功後)
    AdminUser->>Browser: スタッフを検索・フィルタリング、送信対象を選択
    Browser->>WebServer: 検索条件を送信、スタッフリスト取得要求
    WebServer->>Database: 条件に合致するスタッフリスト取得要求
    Database-->>WebServer: スタッフリストを返す
    WebServer->>Browser: スタッフリストを返す
    Browser->>AdminUser: スタッフリストを表示、送信対象を選択
    AdminUser->>Browser: LINEメッセージ送信を選択
    Browser->>WebServer: LINE送信画面表示要求 (送信対象スタッフ情報を含む)
    WebServer->>Browser: LINE送信画面HTMLを返す
    AdminUser->>Browser: メッセージ本文入力、送信ボタンをクリック
    AdminUser->>Browser: LINE送信ボタンをクリック (最終的な送信実行)
    Browser->>WebServer: POST /line/send (メッセージデータ, 送信対象スタッフ情報)
    WebServer->>LineAPI: LINE Messaging API呼び出し (メッセージ送信)
    LineAPI-->>WebServer: 送信結果を返す
    WebServer->>Database: 送信履歴を記録
    Database-->>WebServer: 記録完了を通知
    WebServer->>WebServer: メンテナンスログを記録
    WebServer->>Database: メンテナンスログ保存
    Database-->>WebServer: ログ保存完了
    WebServer->>Browser: 送信結果(成功/失敗)を返す
    Browser->>AdminUser: 送信結果を表示
```

### ③ 認証 ⇒ スタッフ検索・一覧 ⇒ メール送信画面から本文入力し送信

```mermaid
sequenceDiagram
    participant AdminUser as 管理者
    participant Browser
    participant WebServer as Laravel Application
    participant Database
    participant MailService as メール送信サービス

    AdminUser->>Browser: 認証画面にアクセス (認証フローは省略)
    Browser->>WebServer: ...
    WebServer->>Database: ...
    Database-->>WebServer: ...
    WebServer->>Browser: スタッフ検索・一覧画面HTMLを返す (認証成功後)
    AdminUser->>Browser: スタッフを検索・フィルタリング、送信対象を選択
    Browser->>WebServer: 検索条件を送信、スタッフリスト取得要求
    WebServer->>Database: 条件に合致するスタッフリスト取得要求
    Database-->>WebServer: スタッフリストを返す
    WebServer->>Browser: スタッフリストを返す
    Browser->>AdminUser: スタッフリストを表示、送信対象を選択
    AdminUser->>Browser: メール送信を選択
    Browser->>WebServer: メール送信画面表示要求 (送信対象スタッフ情報を含む)
    WebServer->>Browser: メール送信画面HTMLを返す
    AdminUser->>Browser: メール本文入力、送信ボタンをクリック
    Browser->>WebServer: POST /email/send (メールデータ, 送信対象スタッフ情報)
    WebServer->>MailService: メール送信サービス呼び出し
    MailService-->>WebServer: 送信結果を返す
    WebServer->>Database: 送信履歴を記録
    Database-->>WebServer: 記録完了を通知
    WebServer->>WebServer: メンテナンスログを記録
    WebServer->>Database: メンテナンスログ保存
    Database-->>WebServer: ログ保存完了
    WebServer->>Browser: 送信結果(成功/失敗)を返す
    Browser->>AdminUser: 送信結果を表示
```

### ④ 認証 ⇒ スタッフ検索・一覧 ⇒ 新規登録画面からエントリ（スタッフ詳細画面を流用）

```mermaid
sequenceDiagram
    participant AdminUser as 管理者
    participant Browser
    participant WebServer as Laravel Application
    participant Database

    AdminUser->>Browser: 認証画面にアクセス (認証フローは省略)
    Browser->>WebServer: ...
    WebServer->>Database: ...
    Database-->>WebServer: ...
    WebServer->>Browser: スタッフ検索・一覧画面HTMLを返す (認証成功後)
    AdminUser->>Browser: スタッフ検索・一覧画面から「新規登録」を選択
    Browser->>WebServer: 新規登録画面表示要求
    WebServer->>Browser: 新規登録フォームHTMLを返す (スタッフ詳細画面流用)
    AdminUser->>Browser: 新規スタッフ情報入力、登録ボタンクリック
    Browser->>WebServer: POST /staff (登録データ)
    WebServer->>WebServer: 入力値のバリデーション
    alt バリデーション成功
        WebServer->>Database: 新規スタッフ情報の登録を要求
        Database-->>WebServer: 登録完了を通知
        WebServer->>WebServer: メンテナンスログを記録
        WebServer->>Database: メンテナンスログ保存
        Database-->>WebServer: ログ保存完了
        WebServer->>Browser: 登録完了メッセージ、スタッフ一覧へリダイレクト
    else バリデーション失敗
        WebServer->>Browser: エラーメッセージと共に新規登録画面を再表示
    end
```

### ⑤ 認証 ⇒ スタッフ検索・一覧 ⇒ 一括登録（CSVインポート）

```mermaid
sequenceDiagram
    participant AdminUser as 管理者
    participant Browser
    participant WebServer as Laravel Application
    participant Database

    AdminUser->>Browser: 認証画面にアクセス (認証フローは省略)
    Browser->>WebServer: ...
    WebServer->>Database: ...
    Database-->>WebServer: ...
    WebServer->>Browser: スタッフ検索・一覧画面HTMLを返す (認証成功後)
    AdminUser->>Browser: スタッフ検索・一覧画面から「一括登録（CSVインポート）」を選択
    Browser->>WebServer: CSVインポート画面表示要求
    WebServer->>Browser: CSVインポート画面HTMLを返す
    AdminUser->>Browser: CSVファイルを選択、アップロードボタンクリック
    Browser->>WebServer: POST /staff/import (CSVファイル)
    WebServer->>WebServer: CSVファイルの解析とバリデーション
    alt 解析・バリデーション成功
        WebServer->>Database: CSVデータの一括登録を要求
        Database-->>WebServer: 一括登録完了を通知
        WebServer->>WebServer: メンテナンスログを記録
        WebServer->>Database: メンテナンスログ保存
        Database-->>WebServer: ログ保存完了
        WebServer->>Browser: 一括登録結果(成功件数、失敗件数、エラー詳細など)を返す
    else 解析・バリデーション失敗
        WebServer->>Browser: エラー詳細メッセージと共にCSVインポート画面を再表示
    end
    Browser->>AdminUser: 一括登録結果を表示
```
  
## 非同期処理とパフォーマンス最適化

* バッチ送信: LINE一括通知やメール送信はジョブキュー（Redis + Laravel Queue）で非同期実行し、ユーザー操作をブロックしない。
* 進捗表示: 長時間ジョブの進捗はWebSocket（Laravel Echo）でフロントエンドにリアルタイム通知。
* キャッシュ: 頻繁に参照するスタッフ一覧や設定情報はRedisキャッシュを利用し、DB負荷を軽減。
* 負荷試験: k6やJMeterを使用した負荷試験プランを作成し、ピーク時の性能を検証。

## 入力値バリデーションの適用場所

* **フロントエンド（React）**: フォームコンポーネント内で必須チェック、形式チェック、リアルタイムエラー表示を行い、ユーザー体験を向上させます。
* **サーバーサイド（Laravel）**: コントローラーで直接実装するか、FormRequestクラスを利用してリクエスト受信時にバリデーションを実行し、不正データを排除します。

## セキュリティとエラーハンドリング

* CSRF保護: 全てのフォームとAPIエンドポイントにCSRFトークン検証を実装し、不正リクエストを防止。
* セッション管理: セッションタイムアウト、IPアドレス／User-Agentチェックによるセッション固定攻撃対策。
* バックエンドAPIバージョニング: `/api/v1/...` のようにバージョンを明示し、後方互換性を確保。
* エラーレスポンス: 4xx／5xxエラー時に標準化したJSON形式（error_code, message, details）を返却し、クライアントで適切にハンドリング。