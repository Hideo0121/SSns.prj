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

### 9. セキュリティ強化機能の実装

**resources/js/hooks/useSecurityControl.js:**
```javascript
import { useEffect, useCallback } from 'react';
import { useAuth } from '../contexts/AuthContext';

export const useSecurityControl = () => {
    const { user, logout } = useAuth();

    // セキュリティ違反時の処理
    const handleSecurityViolation = useCallback(() => {
        // トークンを破棄
        localStorage.removeItem('auth_token');
        
        // ログアウト処理
        logout();
        
        // エラーページへ遷移
        window.location.href = '/security-error';
    }, [logout]);

    // ページ複製禁止
    const preventPageDuplication = useCallback(() => {
        // 新しいウィンドウ・タブの開始を禁止
        window.addEventListener('beforeunload', (e) => {
            if (user) {
                e.preventDefault();
                e.returnValue = '';
            }
        });

        // 右クリックメニューを禁止
        document.addEventListener('contextmenu', (e) => {
            if (user) {
                e.preventDefault();
            }
        });

        // キーボードショートカット禁止
        document.addEventListener('keydown', (e) => {
            if (user) {
                // Ctrl+N (新しいウィンドウ)
                if (e.ctrlKey && e.key === 'n') {
                    e.preventDefault();
                    handleSecurityViolation();
                }
                // Ctrl+T (新しいタブ)
                if (e.ctrlKey && e.key === 't') {
                    e.preventDefault();
                    handleSecurityViolation();
                }
                // Ctrl+Shift+N (新しいプライベートウィンドウ)
                if (e.ctrlKey && e.shiftKey && e.key === 'N') {
                    e.preventDefault();
                    handleSecurityViolation();
                }
                // F12 (開発者ツール)
                if (e.key === 'F12') {
                    e.preventDefault();
                    handleSecurityViolation();
                }
                // Ctrl+Shift+I (開発者ツール)
                if (e.ctrlKey && e.shiftKey && e.key === 'I') {
                    e.preventDefault();
                    handleSecurityViolation();
                }
            }
        });
    }, [user, handleSecurityViolation]);

    // リロード機能禁止
    const preventReload = useCallback(() => {
        // F5キー、Ctrl+R を禁止
        document.addEventListener('keydown', (e) => {
            if (user) {
                if (e.key === 'F5' || (e.ctrlKey && e.key === 'r')) {
                    e.preventDefault();
                    handleSecurityViolation();
                }
            }
        });

        // ブラウザの戻る/進むボタンを無効化
        window.history.pushState(null, null, window.location.href);
        window.addEventListener('popstate', () => {
            if (user) {
                window.history.pushState(null, null, window.location.href);
                handleSecurityViolation();
            }
        });
    }, [user, handleSecurityViolation]);

    // 非認証状態での認証以外ページ表示禁止
    const preventUnauthorizedAccess = useCallback(() => {
        const currentPath = window.location.pathname;
        const allowedPaths = ['/login', '/security-error'];
        
        if (!user && !allowedPaths.includes(currentPath)) {
            handleSecurityViolation();
        }
    }, [user, handleSecurityViolation]);

    // タブフォーカス監視
    const monitorTabFocus = useCallback(() => {
        let isTabActive = true;
        
        document.addEventListener('visibilitychange', () => {
            if (user) {
                if (document.hidden) {
                    isTabActive = false;
                } else {
                    if (!isTabActive) {
                        // タブが非アクティブだった場合、セキュリティ違反とみなす
                        handleSecurityViolation();
                    }
                    isTabActive = true;
                }
            }
        });

        // ウィンドウフォーカス監視
        window.addEventListener('blur', () => {
            if (user) {
                setTimeout(() => {
                    if (!document.hasFocus()) {
                        handleSecurityViolation();
                    }
                }, 100);
            }
        });
    }, [user, handleSecurityViolation]);

    useEffect(() => {
        preventPageDuplication();
        preventReload();
        preventUnauthorizedAccess();
        monitorTabFocus();

        // クリーンアップ
        return () => {
            window.removeEventListener('beforeunload', () => {});
            document.removeEventListener('contextmenu', () => {});
            document.removeEventListener('keydown', () => {});
            window.removeEventListener('popstate', () => {});
            document.removeEventListener('visibilitychange', () => {});
            window.removeEventListener('blur', () => {});
        };
    }, [preventPageDuplication, preventReload, preventUnauthorizedAccess, monitorTabFocus]);

    return { handleSecurityViolation };
};
```

**resources/js/components/SecurityWrapper.jsx:**
```javascript
import React, { useEffect } from 'react';
import { useSecurityControl } from '../hooks/useSecurityControl';
import { useAuth } from '../contexts/AuthContext';

const SecurityWrapper = ({ children }) => {
    const { user } = useAuth();
    const { handleSecurityViolation } = useSecurityControl();

    useEffect(() => {
        // セキュリティ警告メッセージを表示
        if (user) {
            console.warn('This application is protected by security measures. Any unauthorized access or manipulation will result in immediate logout.');
        }
    }, [user]);

    return <>{children}</>;
};

export default SecurityWrapper;
```

**resources/js/pages/SecurityError.jsx:**
```javascript
import React, { useEffect } from 'react';
import { Box, Typography, Button, Paper } from '@mui/material';
import { Error as ErrorIcon } from '@mui/icons-material';

const SecurityError = () => {
    useEffect(() => {
        // セキュリティエラーページでは履歴を削除
        window.history.replaceState(null, null, '/security-error');
        
        // 戻るボタンを無効化
        window.history.pushState(null, null, '/security-error');
        window.addEventListener('popstate', () => {
            window.history.pushState(null, null, '/security-error');
        });
    }, []);

    const handleReturnToLogin = () => {
        // 全てのローカルストレージを削除
        localStorage.clear();
        sessionStorage.clear();
        
        // ログイン画面へリダイレクト
        window.location.href = '/login';
    };

    return (
        <Box
            sx={{
                minHeight: '100vh',
                display: 'flex',
                alignItems: 'center',
                justifyContent: 'center',
                backgroundColor: '#f5f5f5',
            }}
        >
            <Paper
                elevation={3}
                sx={{
                    padding: 4,
                    textAlign: 'center',
                    maxWidth: 500,
                    width: '90%',
                }}
            >
                <ErrorIcon
                    sx={{
                        fontSize: 80,
                        color: 'error.main',
                        mb: 2,
                    }}
                />
                <Typography variant="h4" gutterBottom color="error">
                    セキュリティエラー
                </Typography>
                <Typography variant="body1" paragraph>
                    セキュリティ違反が検出されました。
                </Typography>
                <Typography variant="body2" paragraph color="text.secondary">
                    • 不正なページアクセス
                    • 認証なしでのページ表示
                    • 禁止された操作の実行
                </Typography>
                <Typography variant="body1" paragraph>
                    セキュリティ保護のため、認証情報が削除されました。
                    <br />
                    再度ログインしてください。
                </Typography>
                <Button
                    variant="contained"
                    color="primary"
                    size="large"
                    onClick={handleReturnToLogin}
                    sx={{ mt: 2 }}
                >
                    ログイン画面へ戻る
                </Button>
            </Paper>
        </Box>
    );
};

export default SecurityError;
```

**resources/js/App.jsx への統合:**
```javascript
import React from 'react';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import { AuthProvider } from './contexts/AuthContext';
import SecurityWrapper from './components/SecurityWrapper';
import ProtectedRoute from './components/ProtectedRoute';
import Login from './pages/Login';
import SecurityError from './pages/SecurityError';
import StaffList from './pages/StaffList';
// ... 他のページコンポーネント

function App() {
    return (
        <AuthProvider>
            <SecurityWrapper>
                <Router>
                    <Routes>
                        <Route path="/login" element={<Login />} />
                        <Route path="/security-error" element={<SecurityError />} />
                        <Route
                            path="/"
                            element={
                                <ProtectedRoute>
                                    <StaffList />
                                </ProtectedRoute>
                            }
                        />
                        {/* 他の保護されたルート */}
                    </Routes>
                </Router>
            </SecurityWrapper>
        </AuthProvider>
    );
}

export default App;
```

**resources/js/components/ProtectedRoute.jsx:**
```javascript
import React from 'react';
import { Navigate } from 'react-router-dom';
import { useAuth } from '../contexts/AuthContext';

const ProtectedRoute = ({ children }) => {
    const { user, loading } = useAuth();

    if (loading) {
        return <div>Loading...</div>;
    }

    if (!user) {
        // 認証されていない場合、セキュリティエラーページへ
        return <Navigate to="/security-error" replace />;
    }

    return children;
};

export default ProtectedRoute;
```
```

### 10. バックエンド側でのセキュリティ強化

**app/Http/Middleware/SecurityMiddleware.php:**
```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class SecurityMiddleware
{
    public function handle(Request $request, Closure $next)
    {
        // 認証が必要なページで認証されていない場合
        if (!Auth::check()) {
            return response()->json([
                'error' => 'Unauthorized access detected',
                'redirect' => '/security-error'
            ], 401);
        }

        // リファラーチェック（直接アクセス防止）
        if (!$request->header('referer') && !$request->is('api/auth/*')) {
            return response()->json([
                'error' => 'Direct access prohibited',
                'redirect' => '/security-error'
            ], 403);
        }

        // User-Agentチェック
        $userAgent = $request->header('User-Agent');
        if (empty($userAgent) || $this->isSuspiciousUserAgent($userAgent)) {
            return response()->json([
                'error' => 'Suspicious access detected',
                'redirect' => '/security-error'
            ], 403);
        }

        return $next($request);
    }

    private function isSuspiciousUserAgent($userAgent)
    {
        $suspiciousPatterns = [
            'bot',
            'crawler',
            'spider',
            'scraper',
            'curl',
            'wget',
            'python',
            'perl',
            'php',
        ];

        $userAgentLower = strtolower($userAgent);
        foreach ($suspiciousPatterns as $pattern) {
            if (strpos($userAgentLower, $pattern) !== false) {
                return true;
            }
        }

        return false;
    }
}
```

**app/Http/Controllers/SecurityController.php:**
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Log;

class SecurityController extends Controller
{
    public function reportViolation(Request $request)
    {
        $user = Auth::user();
        
        Log::warning('Security violation reported', [
            'user_id' => $user ? $user->id : 'guest',
            'user_code' => $user ? $user->user_code : 'N/A',
            'ip_address' => $request->ip(),
            'user_agent' => $request->header('User-Agent'),
            'violation_type' => $request->input('type'),
            'timestamp' => now(),
            'url' => $request->input('url'),
        ]);

        // 現在のユーザーのトークンを全て削除
        if ($user) {
            $user->tokens()->delete();
        }

        // セッションを無効化
        $request->session()->invalidate();
        $request->session()->regenerateToken();

        return response()->json([
            'message' => 'Security violation logged',
            'redirect' => '/security-error'
        ]);
    }
}
```

**routes/web.php への追加:**
```php
// セキュリティ関連ルート
Route::post('/api/security/violation', [App\Http\Controllers\SecurityController::class, 'reportViolation']);
Route::get('/security-error', function () {
    return view('security-error');
});
```

**app/Http/Kernel.php への追加:**
```php
protected $middlewareGroups = [
    'web' => [
        // ... 既存のミドルウェア
    ],
    
    'api' => [
        // ... 既存のミドルウェア
        \App\Http\Middleware\SecurityMiddleware::class,
    ],
];
```

### 11. CSP (Content Security Policy) 設定

**app/Http/Middleware/ContentSecurityPolicyMiddleware.php:**
```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class ContentSecurityPolicyMiddleware
{
    public function handle(Request $request, Closure $next)
    {
        $response = $next($request);

        $csp = [
            "default-src 'self'",
            "script-src 'self' 'unsafe-inline'",
            "style-src 'self' 'unsafe-inline' https://fonts.googleapis.com",
            "font-src 'self' https://fonts.gstatic.com",
            "img-src 'self' data:",
            "connect-src 'self'",
            "frame-ancestors 'none'",
            "base-uri 'self'",
            "form-action 'self'",
            "upgrade-insecure-requests",
        ];

        $response->headers->set('Content-Security-Policy', implode('; ', $csp));
        $response->headers->set('X-Frame-Options', 'DENY');
        $response->headers->set('X-Content-Type-Options', 'nosniff');
        $response->headers->set('X-XSS-Protection', '1; mode=block');
        $response->headers->set('Referrer-Policy', 'strict-origin-when-cross-origin');

        return $response;
    }
}
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

