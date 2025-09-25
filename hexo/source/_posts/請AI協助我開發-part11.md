---
title: 請AI協助我開發-part11
date: 2025-09-22 15:08:53
tags:  [AI , back-end]
---

# login 登入程序的選擇

常見的登入驗證中我選擇使用Token-Based Authentication（JWT 為主流）

* 流程

使用者輸入帳密 → 後端簽發 JWT
前端儲存在 localStorage / Cookie
後續請求在 Authorization: Bearer <token> 傳給後端
後端驗證簽名與有效期即可

* 優點

無狀態（Stateless），適合 API、微服務
跨平台（Web、iOS、Android 都能用）

* 缺點

Token 洩漏即遭濫用（需搭配 HTTPS、過期機制）
Token 無法輕易註銷（需額外黑名單機制）


## 安裝套件
用 tymon/jwt-auth：
```
composer require tymon/jwt-auth
```
安裝完成後，發佈設定檔：
```
php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\LaravelServiceProvider"
```
這會在 config/jwt.php 生成設定檔。

接著生成 JWT 秘鑰
```
php artisan jwt:secret
```
這會在 .env 加入一行：
```
JWT_SECRET=隨機字串
```

## 修改 User Model
```
<?php

namespace App\Models;

// use Illuminate\Contracts\Auth\MustVerifyEmail;
// 認證相關的基底類
use Illuminate\Foundation\Auth\User as Authenticatable;
// 通知相關
use Illuminate\Notifications\Notifiable;
// 假資料工廠
use Illuminate\Database\Eloquent\Factories\HasFactory;
// 加上JWT 相關介面
use Tymon\JWTAuth\Contracts\JWTSubject;


class User extends Authenticatable implements JWTSubject //後面加上 implements JWTSubject
{
    // 加上JWTSubject 需要實作兩個方法
    public function getJWTIdentifier()
    {
        return $this->getKey(); // 通常是 id
    }

    public function getJWTCustomClaims()
    {
        return [];
    }
}
```
Model原本有的內容依舊保留，只是增加一些實作JWT需要的方法
## 建立 AuthController

我把user的密碼欄位名稱改成password，後續操作會單純很多，因為：
Laravel 預設認密碼欄位就是 password

在 app/Http/Controllers 建立 AuthController.php，並且裡面有基本的三個方法login,me,logout

```
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\User;
use Illuminate\Support\Facades\Hash;
use Tymon\JWTAuth\Facades\JWTAuth;

class AuthController extends Controller
{
    // 登入
    public function login(Request $request)
    {
        $credentials = $request->validate([
            'user_id' => 'required|string',
            'password' => 'required|string',
        ]);

        try {
            // 嘗試登入並產生 JWT token
            if (!$token = JWTAuth::attempt($credentials)) {
                return response()->json(['error' => 'Invalid credentials'], 401);
            }
        } catch (JWTException $e) {
            return response()->json(['error' => 'Could not create token'], 500);
        }

        // 登入成功，回傳 token
        return response()->json([
            'access_token' => $token,
            'token_type' => 'bearer',
            'expires_in' => JWTAuth::factory()->getTTL() * 60, // 這樣才會用 JWT TTL
        ]);
    }

    /**
     * 取得當前使用者資料
     */
    public function me()
    {
        try {
            $user = JWTAuth::parseToken()->authenticate();
        } catch (\Exception $e) {
            return response()->json(['error' => 'Token invalid or expired'], 401);
        }

        return response()->json($user);
    }
    /**
     * 使用者登出（將 token 失效）
     */
    public function logout(Request $request)
    {
        $token = $request->bearerToken();
        if ($token) {
            JWTAuth::invalidate($token);
        }

        return response()->json(['message' => 'Successfully logged out']);
    }
}
```
這時我就有問題，
1. 這樣API回傳的格式不統一，
2. 前端要打兩次API才能拿到使用者的資訊

愛醬也認同我的疑惑，這會造成前端處理上不太一致，因為可能要先判斷有沒有 access_token 才知道成功或失敗。
並建議做些調整，統一回傳一個固定結構
像是這樣
```
{
    "success": true/false,
    "data": {...},      // 成功時放 token + user
    "message": "描述文字"
}
```
修正後的AuthController.php
```
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;
use Tymon\JWTAuth\Facades\JWTAuth;

class AuthController extends Controller
{
    // 登入

    public function login(Request $request)
    {
        $credentials = $request->validate([
            'user_id' => 'required|string',
            'password' => 'required|string',
        ]);

        try {
            // 嘗試登入並產生 JWT token
            if (! $token = JWTAuth::attempt($credentials)) {
                return response()->json([
                    'success' => false,
                    'data' => null,
                    'message' => 'Invalid credentials',
                ], 401);
            }
        } catch (JWTException $e) {
            return response()->json([
                'success' => false,
                'data' => null,
                'message' => 'Could not create token',
            ], 500);
        }

        // 登入成功，回傳 token + user
        $user = auth()->user(); // 取得登入的使用者

        return response()->json([
            'success' => true,
            'data' => [
                'access_token' => $token,
                'token_type' => 'bearer',
                'expires_in' => JWTAuth::factory()->getTTL() * 60,
                'user' => $user,
            ],
            'message' => 'Login successful',
        ]);
    }

    /**
     * 取得當前使用者資料
     */
    public function me()
    {
        try {
            $user = JWTAuth::parseToken()->authenticate();

            return response()->json([
                'success' => true,
                'data' => $user,
                'message' => 'User fetched successfully',
            ]);
        } catch (\Tymon\JWTAuth\Exceptions\TokenExpiredException $e) {
            return response()->json([
                'success' => false,
                'message' => 'Token has expired',
            ], 401);
        } catch (\Tymon\JWTAuth\Exceptions\TokenInvalidException $e) {
            return response()->json([
                'success' => false,
                'message' => 'Token is invalid',
            ], 401);
        } catch (\Tymon\JWTAuth\Exceptions\JWTException $e) {
            return response()->json([
                'success' => false,
                'message' => 'Token not provided',
            ], 401);
        }
    }

    /**
     * 使用者登出（將 token 失效）
     */
    public function logout(Request $request)
    {
        $token = $request->bearerToken();
        if ($token) {
            JWTAuth::invalidate($token); //把 token 加到 黑名單 (blacklist)，讓它即使還沒過期也不能再用
        }

        return response()->json(['message' => 'Successfully logged out']);
    }

}
```
針對fuction me 跟 logout 解說
## function me 獲取使用者資料
核心的內容是這行
```
$user = JWTAuth::parseToken()->authenticate();
```
* parseToken()：會從 request header 的 Authorization: Bearer <token> 裡自動取出 JWT。
* authenticate()：解碼 JWT → 確認 token 是否有效、是否過期 → 如果正常就回傳對應的使用者。
👉 所以這裡已經隱含了 token 驗證。如果 token 無效/過期，會直接丟例外 (TokenExpiredException / TokenInvalidException)。

JWT（JSON Web Token）本身就包含了 使用者的識別資訊，
在解析 Token之後可以讀取 Payload，之後載入使用者資訊

## fuction logout 使用者登出

登出是要 讓 token 立即失效。
JWT 本質上是「無狀態」的，理論上只要簽發就會一直有效到過期時間為止。
invalidate($token) 的意思是把 token 加到 黑名單 (blacklist)，讓它即使還沒過期也不能再用。
所以這裡要手動拿出 token 丟給 invalidate()。

## fuction register 使用者註冊

之後加上使用者註冊的功能
```
    /**
     * 使用者註冊
     */
    public function register(Request $request)
    {
        $validated = $request->validate([
            'user_id' => 'required|string|max:50|unique:users,user_id',
            'password' => 'required|string|min:6',
            'nickname' => 'required|string|max:50',
            'weight' => 'nullable|numeric|min:0',
            'status' => 'required|string|max:50',
        ]);

        try {
            // 建立使用者
            $user = User::create([
                'user_id' => $validated['user_id'],
                'password' => bcrypt($validated['password']), // bcrypt 加密
                'nickname' => $validated['nickname'],
                'weight' => $validated['weight'] ?? 60.0, // 預設體重 60.0
                'status' => "active",
                'last_login_at' => null,
                'password_change_at' => now(), // 註冊時預設密碼設定時間
            ]);

            // 註冊完成馬上登入並回傳 token
            $token = JWTAuth::fromUser($user);

            return response()->json([
                'success' => true,
                'data' => [
                    'user' => $user,
                    'access_token' => $token,
                    'token_type' => 'bearer',
                    'expires_in' => JWTAuth::factory()->getTTL() * 60,
                ],
                'message' => 'User registered successfully',
            ], 201);

        } catch (\Exception $e) {
            return response()->json([
                'success' => false,
                'message' => 'Registration failed',
                'error' => $e->getMessage(),
            ], 500);
        }
    }
```

## updateProfile 使用者更新資料

常見的 帳號生命週期 API包含下面幾個
✅ login
✅ register
✅ me
✅ logout
✅ updateProfile
```
    /**
     * 更新使用者資料
     */
    public function updateProfile(Request $request)
    {
        try {
            $user = JWTAuth::parseToken()->authenticate();

        $validated = $request->validate([
            'nickname' => 'nullable|string|max:50',
            'weight'   => 'nullable|numeric|min:0',
            'password' => 'nullable|string|min:6|confirmed', // confirmed 會檢查 password_confirmation
        ]);
        if (isset($validated['nickname'])) {
            $user->nickname = $validated['nickname'];
        }

        if (isset($validated['weight'])) {
            $user->weight = $validated['weight'];
        }

        if (isset($validated['password'])) {
            $user->password = bcrypt($validated['password']);
            $user->password_change_at = now();
        }

        $user->save();
        } catch (\Tymon\JWTAuth\Exceptions\TokenExpiredException $e) {
            return response()->json([
                'success' => false,
                'message' => 'Token has expired',
            ], 401);
        } catch (\Tymon\JWTAuth\Exceptions\TokenInvalidException $e) {
            return response()->json([
                'success' => false,
                'message' => 'Token is invalid',
            ], 401);
        } catch (\Tymon\JWTAuth\Exceptions\JWTException $e) {
            return response()->json([
                'success' => false,
                'message' => 'Token not provided',
            ], 401);
        }
    }
```
這個做法是將其他個人資料更新與密碼更新放在一起，雖然簡單方便但是安全性我就不是很滿意，也跟常接觸的修該資料有點不一樣

有些人會選擇個人資料更新與更新密碼分兩個API執行
讓密碼更新的API需要輸入舊密碼增加安全性
或是所有更新都需要輸入密碼，但是會讓使用者體驗極差
我想想能不能僅有敏感資訊需要輸入密碼，其他直接更新就好
```
    public function updateProfile(Request $request)
    {
        try {
            $user = JWTAuth::parseToken()->authenticate();

        $validated = $request->validate([
            'nickname' => 'nullable|string|max:50',
            'weight'   => 'nullable|numeric|min:0',
            'password' => 'nullable|string|min:6', // 新密碼
            'old_password' => 'required_with:password|string', // 當要改密碼時，舊密碼必填
        ]);

            // 如果要更新密碼 → 驗證舊密碼
            if ($request->has('password')) {
                if (! Hash::check($request->input('old_password'), $user->password)) {
                    rreturn response()->json([
                    'success' => false,
                    'message' => '舊密碼錯誤',
                ], 403);
                }
                $user->password = bcrypt($request->input('password'));
                $user->password_change_at = now();
            }

        // 更新其他欄位
        if (isset($validated['nickname'])) {
            $user->nickname = $validated['nickname'];
        }

        if (isset($validated['weight'])) {
            $user->weight = $validated['weight'];
        }
            $user->save();

            return response()->json([
                'success' => true,
                'message' => '資料更新成功',
                'data' => $user,
            ]);
        } catch (\Tymon\JWTAuth\Exceptions\TokenExpiredException $e) {
            return response()->json([
                'success' => false,
                'message' => 'Token has expired',
            ], 401);
        } catch (\Tymon\JWTAuth\Exceptions\TokenInvalidException $e) {
            return response()->json([
                'success' => false,
                'message' => 'Token is invalid',
            ], 401);
        } catch (\Tymon\JWTAuth\Exceptions\JWTException $e) {
            return response()->json([
                'success' => false,
                'message' => 'Token not provided',
            ], 401);
        }
    }
```
雖然運用fill()批量賦值處理(一次把陣列中的多個欄位丟給 model)。
簡潔、好維護、適合更新多個欄位時，不用一個個寫 if 判斷。
但是安全性優先 → 用 isset() 一個個更新欄位，我最還是選擇使用這個

# AuthController 完成之後，設定 Route

現在有了完整的 後端 JWT 認證流程，
接著確保的 routes/api.php 有對應的 API 路由

才能再呼叫api的時候有反應
```
use App\Http\Controllers\AuthController;

Route::post('/login', [AuthController::class, 'login']);
Route::post('/register', [AuthController::class, 'register']);

// 需要 JWT 驗證的路由
Route::middleware('auth:api')->group(function () {
    Route::get('/me', [AuthController::class, 'me']);
    Route::post('/logout', [AuthController::class, 'logout']);
    Route::put('/profile', [AuthController::class, 'updateProfile']);
});
```

## 設定 JWT middleware，

確認 config/auth.php 裡的 guards 有設定
```
    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],
        // 新增一個 'api' guard，使用 'jwt' 驅動
        'api' => [
            'driver' => 'jwt',
            'provider' => 'users',
            'hash' => false,
        ],
    ],
```
這樣 auth()->user() 才會在 API 中拿到 JWT 對應的使用者。

之後可能要找時間研究laravel命名慣例，這次在命名上吃了不少苦頭