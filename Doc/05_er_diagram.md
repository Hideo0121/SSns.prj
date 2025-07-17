# ER図

このセクションでは、アプリケーションのデータベース構造を表すエンティティリレーションシップ図 (ER図) を記述します。これは、データの関連性や整合性を理解するために重要です。

### 主要エンティティの例

```mermaid
erDiagram
    %% エンティティ定義
    USERS {
        int id PK
        string user_code UK
        string name
        string email UK
        string password_hash "ハッシュ化されたパスワード"
        string role "全権管理者/一般管理者/スタッフ"
        string phone_number "電話番号"
        string mobile_phone_number "携帯電話番号"
        string avatar_photo "顔写真"
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

    EMAIL_BROADCAST_MESSAGES {
        int id PK
        int admin_user_id FK
        string subject
        string body
        datetime sent_at
        string status
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
    USERS ||--o{ EMAIL_BROADCAST_MESSAGES : "sends"
    USERS ||--o{ MAINTENANCE_LOGS : "performs"
    
    THREADS ||--o{ COMMENTS : "has"
    THREADS ||--o{ FAVORITES : "can be favorited"
    THREADS }o--|| CATEGORIES : "belongs to"
    
    EMAIL_BROADCAST_MESSAGES ||--o{ EMAIL_BROADCAST_TARGETS : "includes"
    
    USERS ||--o{ EMAIL_BROADCAST_TARGETS : "targeted by"
```

## エンティティ説明

**USERS**: システムのメンバー情報。role には「全権管理者」「一般管理者」「スタッフ」の分類を保持します。phone_number (電話番号) と mobile_phone_number (携帯電話番号) を追加しました。
avatar_photo フィールドでメンバーの顔写真を保持します。
**セキュリティ**: password_hash フィールドにはbcryptアルゴリズムでハッシュ化されたパスワードを保存し、平文パスワードは保存しません。user_code フィールドにユニーク制約を追加し、認証に使用します。

**MESSAGES**: 個別メッセージの送受信履歴。

**THREADS**: 掲示板のトピック。

**COMMENTS**: 掲示板のトピックに対するコメント。

**CATEGORIES**: 掲示板トピックのカテゴリ。

**EMAIL_BROADCAST_MESSAGES**: メール一括送信メッセージの履歴。どの管理者が送信したか(admin_user_id)、送信結果(status)などを記録します。

**EMAIL_BROADCAST_TARGETS**: メール一括送信メッセージが誰に送られたかを記録する中間テーブル（多対多のリレーション）。

**MAINTENANCE_LOGS**: システムのメンテナンス履歴を記録するログテーブル。誰が(user_code)、いつ(created_at)、何を(action_type)、どのデータ(target_table, target_id)に対して、どのように(old_data, new_data)操作したかを記録します。操作元のIPアドレスも記録することで、セキュリティ監査に役立てます。

**FAVORITES**: ユーザーが掲示板の記事をお気に入り登録した履歴を管理するテーブル。user_code と thread_id の組み合わせで、どのユーザーがどの記事をお気に入り登録したかを記録します。
