# 技術スタックとセットアップ方法

## 技術スタック

* **バックエンド**: **PHP + Laravel**
* **認証**: **Laravel Breeze** + **Laravel Sanctum** を使用してuser_codeとpasswordによる認証を実装
* **フロントエンド**: **Tailwind CSS** と **React** を利用し、動的でインタラクティブなページを構築します。
* **UIコンポーネント**: テキスト、ラベル、ボタンなどのUIパーツには **Material-UI** を利用します。
* **データベース**: **MySQL** を使用します。
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
3.  **Laravel Breezeのインストール:**
    ```bash
    composer require laravel/breeze --dev
    php artisan breeze:install react
    ```
4.  **Laravel Sanctumのインストール:**
    ```bash
    composer require laravel/sanctum
    php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
    php artisan migrate
    ```
5.  **JavaScript依存関係のインストール (フロントエンドにReactを使用するため):**
    ```bash
    npm install # または yarn install
    ```
6.  **.envファイルの作成と設定:**
    ```bash
    cp .env.example .env
    ```
    `.env`ファイルを開き、データベース接続情報とメール送信サービス情報を設定してください。
    ```env
    DB_CONNECTION=mysql
    DB_HOST=127.0.0.1
    DB_PORT=3306
    DB_DATABASE=your_database_name
    DB_USERNAME=your_username
    DB_PASSWORD=your_password

    # Sanctum Configuration
    SANCTUM_STATEFUL_DOMAINS=localhost:3000,127.0.0.1:8000
    SESSION_DRIVER=cookie
    SESSION_DOMAIN=localhost

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
7.  **アプリケーションキーの生成:**
    ```bash
    php artisan key:generate
    ```
8.  **データベースマイグレーションの実行:**
    ```bash
    php artisan migrate
    ```
9.  **フロントエンドのビルド (開発用):**
    ```bash
    npm run dev # または yarn dev
    ```
10. **開発サーバーの起動:**
    ```bash
    php artisan serve
    ```
    ブラウザで `http://127.0.0.1:8000` にアクセスして、アプリケーションを確認してください。

## 認証システムの設定 (Laravel Breeze + Sanctum)

Laravel Breezeインストール後、user_codeでの認証とSanctumによるAPIトークン認証に対応するため以下の設定を行います：

### 1. Laravel Sanctumの設定

**config/sanctum.php の編集:**
```php
<?php

return [
    'stateful' => explode(',', env('SANCTUM_STATEFUL_DOMAINS', 'localhost,localhost:3000,127.0.0.1,127.0.0.1:8000,::1')),
    'guard' => ['web'],
    'expiration' => 180, // 3時間 (分単位)
    'middleware' => [
        'verify_csrf_token' => App\Http\Middleware\VerifyCsrfToken::class,
        'encrypt_cookies' => App\Http\Middleware\EncryptCookies::class,
    ],
];
```

**app/Http/Kernel.php の編集:**
```php
'api' => [
    \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
    'throttle:api',
    \Illuminate\Routing\Middleware\SubstituteBindings::class,
],
```

### 2. Userモデルの修正

**app/Models/User.php:**
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;

    protected $fillable = [
        'user_code',
        'name',
        'email',
        'password',
    ];

    protected $hidden = [
        'password',
        'remember_token',
    ];

    protected $casts = [
        'email_verified_at' => 'datetime',
    ];

    // user_codeでの認証を有効にする
    public function getAuthIdentifierName()
    {
        return 'user_code';
    }

    // user_codeでユーザーを検索
    public function findForPassport($identifier)
    {
        return $this->where('user_code', $identifier)->first();
    }
}
```

### 3. 認証設定の変更

**config/auth.php:**
```php
'providers' => [
    'users' => [
        'driver' => 'eloquent',
        'model' => App\Models\User::class,
    ],
],
```

### 4. マイグレーションの追加

**database/migrations/xxxx_add_user_code_to_users_table.php:**
```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up()
    {
        Schema::table('users', function (Blueprint $table) {
            $table->string('user_code')->unique()->after('id');
        });
    }

    public function down()
    {
        Schema::table('users', function (Blueprint $table) {
            $table->dropColumn('user_code');
        });
    }
};
```

### 5. 認証コントローラーの修正

**app/Http/Controllers/Auth/AuthenticatedSessionController.php:**
```php
<?php

namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use App\Http\Requests\Auth\LoginRequest;
use Illuminate\Http\Request;
use Illuminate\Http\Response;
use Illuminate\Support\Facades\Auth;

class AuthenticatedSessionController extends Controller
{
    public function store(LoginRequest $request): Response
    {
        $request->authenticate();

        $request->session()->regenerate();

        $user = Auth::user();
        
        // Sanctumトークンの発行
        $token = $user->createToken('auth-token', ['*'], now()->addHours(3));

        return response()->json([
            'user' => $user,
            'token' => $token->plainTextToken,
            'expires_at' => $token->accessToken->expires_at,
        ]);
    }

    public function destroy(Request $request): Response
    {
        // 現在のトークンを削除
        $request->user()->currentAccessToken()->delete();

        Auth::guard('web')->logout();

        $request->session()->invalidate();
        $request->session()->regenerateToken();

        return response()->noContent();
    }
}
```

### 6. ログインリクエストの修正

**app/Http/Requests/Auth/LoginRequest.php:**
```php
<?php

namespace App\Http\Requests\Auth;

use Illuminate\Auth\Events\Lockout;
use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\RateLimiter;
use Illuminate\Support\Str;
use Illuminate\Validation\ValidationException;

class LoginRequest extends FormRequest
{
    public function rules()
    {
        return [
            'user_code' => ['required', 'string'],
            'password' => ['required', 'string'],
        ];
    }

    public function authenticate()
    {
        $this->ensureIsNotRateLimited();

        if (! Auth::attempt(['user_code' => $this->user_code, 'password' => $this->password], $this->boolean('remember'))) {
            RateLimiter::hit($this->throttleKey());

            throw ValidationException::withMessages([
                'user_code' => trans('auth.failed'),
            ]);
        }

        RateLimiter::clear($this->throttleKey());
    }

    public function ensureIsNotRateLimited()
    {
        if (! RateLimiter::tooManyAttempts($this->throttleKey(), 5)) {
            return;
        }

        event(new Lockout($this));

        $seconds = RateLimiter::availableIn($this->throttleKey());

        throw ValidationException::withMessages([
            'user_code' => trans('auth.throttle', [
                'seconds' => $seconds,
                'minutes' => ceil($seconds / 60),
            ]),
        ]);
    }

    public function throttleKey()
    {
        return Str::transliterate(Str::lower($this->input('user_code')).'|'.$this->ip());
    }
}
```

### 7. APIルートの設定

**routes/api.php:**
```php
<?php

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
    return $request->user();
});

Route::middleware('auth:sanctum')->group(function () {
    Route::get('/staff', [App\Http\Controllers\StaffController::class, 'index']);
    Route::post('/staff', [App\Http\Controllers\StaffController::class, 'store']);
    Route::get('/staff/{id}', [App\Http\Controllers\StaffController::class, 'show']);
    Route::put('/staff/{id}', [App\Http\Controllers\StaffController::class, 'update']);
    Route::delete('/staff/{id}', [App\Http\Controllers\StaffController::class, 'destroy']);
    
    Route::post('/mail/send', [App\Http\Controllers\MailController::class, 'send']);
    Route::post('/auth/refresh', [App\Http\Controllers\Auth\AuthenticatedSessionController::class, 'refresh']);
});
```

### 8. フロントエンド (React) での実装

**resources/js/lib/api.js:**
```javascript
import axios from 'axios';

const api = axios.create({
    baseURL: '/api',
    withCredentials: true,
    headers: {
        'Content-Type': 'application/json',
        'Accept': 'application/json',
    },
});

// トークンを localStorage から取得して Authorization ヘッダーに設定
api.interceptors.request.use((config) => {
    const token = localStorage.getItem('auth_token');
    if (token) {
        config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
});

// 401 エラーの場合、ログイン画面にリダイレクト
api.interceptors.response.use(
    (response) => response,
    (error) => {
        if (error.response?.status === 401) {
            localStorage.removeItem('auth_token');
            window.location.href = '/login';
        }
        return Promise.reject(error);
    }
);

export default api;
```

**resources/js/contexts/AuthContext.jsx:**
```javascript
import React, { createContext, useContext, useEffect, useState } from 'react';
import api from '../lib/api';

const AuthContext = createContext();

export const useAuth = () => {
    const context = useContext(AuthContext);
    if (!context) {
        throw new Error('useAuth must be used within an AuthProvider');
    }
    return context;
};

export const AuthProvider = ({ children }) => {
    const [user, setUser] = useState(null);
    const [loading, setLoading] = useState(true);

    useEffect(() => {
        const token = localStorage.getItem('auth_token');
        if (token) {
            api.get('/user')
                .then(response => {
                    setUser(response.data);
                })
                .catch(() => {
                    localStorage.removeItem('auth_token');
                })
                .finally(() => {
                    setLoading(false);
                });
        } else {
            setLoading(false);
        }
    }, []);

    const login = async (userCode, password) => {
        try {
            const response = await api.post('/login', {
                user_code: userCode,
                password: password,
            });
            
            localStorage.setItem('auth_token', response.data.token);
            setUser(response.data.user);
            
            // 自動更新のスケジュール
            scheduleTokenRefresh(response.data.expires_at);
            
            return response.data;
        } catch (error) {
            throw error;
        }
    };

    const logout = async () => {
        try {
            await api.post('/logout');
        } catch (error) {
            console.error('Logout error:', error);
        } finally {
            localStorage.removeItem('auth_token');
            setUser(null);
        }
    };

    const scheduleTokenRefresh = (expiresAt) => {
        const now = new Date();
        const expires = new Date(expiresAt);
        const refreshTime = expires.getTime() - now.getTime() - (30 * 60 * 1000); // 30分前

        if (refreshTime > 0) {
            setTimeout(() => {
                refreshToken();
            }, refreshTime);
        }
    };

    const refreshToken = async () => {
        try {
            const response = await api.post('/auth/refresh');
            localStorage.setItem('auth_token', response.data.token);
            scheduleTokenRefresh(response.data.expires_at);
        } catch (error) {
            console.error('Token refresh failed:', error);
            logout();
        }
    };

    const value = {
        user,
        login,
        logout,
        loading,
    };

    return (
        <AuthContext.Provider value={value}>
            {children}
        </AuthContext.Provider>
    );
};
```

## 認証システムのカスタマイズ

Laravel Breezeインストール後、user_codeでの認証に対応するため以下の設定を行います：

1. **認証設定の変更 (config/auth.php)**
   ```php
   'providers' => [
       'users' => [
           'driver' => 'eloquent',
           'model' => App\Models\User::class,
           'username' => 'user_code', // emailからuser_codeに変更
       ],
   ],
   ```

2. **Userモデルの修正**
   - `user_code`フィールドを追加
   - `fillable`配列にuser_codeを追加
   - 必要に応じてバリデーションルールを調整

3. **認証フォームの修正**
   - ログイン画面のemailフィールドをuser_codeに変更
   - フロントエンドコンポーネントの修正

---

## 前提条件

- PHP 8.1 以上
- Composer 2.x
- Node.js 16.x またはそれ以上
- MySQL 5.7 以上または MariaDB 10.2 以上
- Redis (ジョブキュー、キャッシュ用)

## CI/CD & デプロイ

- **GitHub Actions** で以下を自動化:
  - プッシュ時のLintチェック (`phpstan`, `eslint`, `stylelint`)
  - プルリクエスト作成時のユニットテストおよび統合テスト (`phpunit`)
  - `main` マージ時のDockerイメージビルドとContainer Registryへのプッシュ
  - `staging`/`production` ブランチマージ時のデプロイジョブ実行
- **デプロイ環境**: Docker Compose または Kubernetes 上のコンテナ
  - Docker Compose ファイルにより、`app`, `web`(nginx), `db`, `redis` を構成
  - GitHub Container Registry などのプライベートレジストリにイメージをプッシュ
  - Docker Secrets または `.env.prod` による環境変数管理
  - 本番環境では、`docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d --build` を実行

## 環境変数とシークレット管理

- Gitリポジトリには `.env` をコミットしない

