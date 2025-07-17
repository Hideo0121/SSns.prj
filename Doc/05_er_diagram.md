# ER図

このセクションでは、アプリケーションのデータベース構造を表すエンティティリレーションシップ図 (ER図) を記述します。これは、データの関連性や整合性を理解するために重要です。

### 主要エンティティの例

```mermaid
erDiagram
    %% エンティティ定義
    USERS {
        int id PK
        string user_code
        string name
        string email UK
        string password
        string line_user_id "LINEアカウント紐付け用"
        string role "全権管理者/一般管理者/スタッフ"
        string phone_number "電話番号"
        string mobile_phone_number "携帯電話番号"
        string avatar_photo "顔写真"
        string pc_setup_photo "PC設置環境写真"
        string security_software_photo "セキュリティソフト稼働写真"
        string pc_logon_screenshot "PCログオン画面"
        string id_card_photo "身分証明書写真"
        datetime created_at
        datetime updated_at
    }

    MESSAGES {
        int id PK
        int sender_id FK
        int receiver_id FK
        string content
        datetime created_at
        datetime updated_at
    }

    THREADS {
        int id PK
        int user_code FK
        string title
        string content
        int category_id FK
        datetime created_at
        datetime updated_at
    }

    COMMENTS {
        int id PK
        int thread_id FK
        int user_code FK
        string content
        datetime created_at
        datetime updated_at
    }

    CATEGORIES {
        int id PK
        string name
        datetime created_at
        datetime updated_at
    }

    LINE_BROADCAST_MESSAGES {
        int id PK
        int admin_user_id FK
        string message_content
        datetime sent_at
        string status
        string line_response_id
    }

    EMAIL_BROADCAST_MESSAGES {
        int id PK
        int admin_user_id FK
        string subject
        string body
        datetime sent_at
        string status
    }

    LINE_BROADCAST_TARGETS {
        int broadcast_id FK
        int user_code FK
    }

    EMAIL_BROADCAST_TARGETS {
        int broadcast_id FK
        int user_code FK
    }

    MAINTENANCE_LOGS {
        int id PK
        int user_code FK
        string action_type
        string target_table
        int target_id
        text old_data
        text new_data
        string ip_address
        datetime created_at
    }

    FAVORITES {
        int id PK
        int user_code FK
        int thread_id FK
        datetime created_at
    }

    %% 関係性定義
    USERS ||--o{ MESSAGES : "sends/receives"
    USERS ||--o{ THREADS : "creates"
    USERS ||--o{ COMMENTS : "posts"
    USERS ||--o{ FAVORITES : "has"
    USERS ||--o{ LINE_BROADCAST_MESSAGES : "sends"
    USERS ||--o{ EMAIL_BROADCAST_MESSAGES : "sends"
    USERS ||--o{ MAINTENANCE_LOGS : "performs"
    
    THREADS ||--o{ COMMENTS : "has"
    THREADS ||--o{ FAVORITES : "can be favorited"
    THREADS }o--|| CATEGORIES : "belongs to"
    
    LINE_BROADCAST_MESSAGES ||--o{ LINE_BROADCAST_TARGETS : "includes"
    EMAIL_BROADCAST_MESSAGES ||--o{ EMAIL_BROADCAST_TARGETS : "includes"
    
    USERS ||--o{ LINE_BROADCAST_TARGETS : "targeted by"
    USERS ||--o{ EMAIL_BROADCAST_TARGETS : "targeted by"
```

## エンティティ説明

**USERS**: システムのメンバー情報。line_user_id はLINEアカウントとの紐付けに利用します。role には「全権管理者」「一般管理者」「スタッフ」の分類を保持します。phone_number (電話番号) と mobile_phone_number (携帯電話番号) を追加しました。
さらに、メンバーの顔写真、PC設置環境写真、セキュリティソフト稼働写真、PCログオン画面のスクリーンショット、身分証明書写真を保持するフィールドを追加しました。

**MESSAGES**: 個別メッセージの送受信履歴。

**THREADS**: 掲示板のトピック。

**COMMENTS**: 掲示板のトピックに対するコメント。

**CATEGORIES**: 掲示板トピックのカテゴリ。

**LINE_BROADCAST_MESSAGES**: LINE一括送信メッセージの履歴。どの管理者が送信したか(admin_user_id)、送信結果(status)、LINEからのレスポンスIDなどを記録します。

**EMAIL_BROADCAST_MESSAGES**: メール一括送信メッセージの履歴。同様に送信した管理者や送信結果を記録します。

**LINE_BROADCAST_TARGETS**: LINE一括送信メッセージが誰に送られたかを記録する中間テーブル（多対多のリレーション）。

**EMAIL_BROADCAST_TARGETS**: メール一括送信メッセージが誰に送られたかを記録する中間テーブル（多対多のリレーション）。

**MAINTENANCE_LOGS**: システムのメンテナンス履歴を記録するログテーブル。誰が(user_code)、いつ(created_at)、何を(action_type)、どのデータ(target_table, target_id)に対して、どのように(old_data, new_data)操作したかを記録します。操作元のIPアドレスも記録することで、セキュリティ監査に役立てます。

**FAVORITES**: ユーザーが掲示板の記事をお気に入り登録した履歴を管理するテーブル。user_code と thread_id の組み合わせで、どのユーザーがどの記事をお気に入り登録したかを記録します。
