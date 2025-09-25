---
title: 請AI協助我開發-part10
date: 2025-09-11 12:52:35
tags: [AI , back-end]
---


# Seeder建立資料

之後就是建立測試用資料
```
// 終端機
php artisan make:seeder UserSeeder
php artisan make:factory UserFactory
php artisan db:seed
```

這部分有三個指令
* php artisan make:seeder UserSeeder
➡️ 建立一個 Seeder 類別，用來定義要塞進資料庫的「假資料」。
產生後會在 database/seeders/UserSeeder.php 裡面，通常用來建立第一筆的管理者資料
* php artisan make:factory UserFactory
➡️ 建立一個 Factory 類別，用來自動生成大量的假資料（通常搭配 Faker 套件）。
* php artisan db:seed
➡️ 執行 所有 Seeder（或指定 Seeder）來把資料寫進資料庫。

這是我建立第一筆資料的Seeder
```
<?php

namespace Database\Seeders;

use Illuminate\Database\Console\Seeds\WithoutModelEvents;
use Illuminate\Database\Seeder;

class UserSeeder extends Seeder
{
    /**
     * Run the database seeds.
     */
    public function run(): void
    {
        User::create([
            'user_id' => 'niki',
            'password_hash' => bcrypt('123456'),
            'nickname' => '烏龜',
            'weight' => 77.5,
            'status' => 'active',
            'last_login_at' => now(),
            'password_change_at' => now(),
        ]);
    }
}
```

這時候出現了意外的插曲，時間與我建立的時間不同
時間差了8小時，感覺應該是時區設定問題

愛醬表示我推測的沒錯是config/app.php的時區設定問題
將
'timezone' => 'UTC',
改為
'timezone' => 'Asia/Taipei',

這樣建立的資料時間就是我指定時區的時間了

# Controller : 處理使用者請求（Web 或 API），調用 Model 做資料庫操作，最後回傳結果

透過artisan 生成Controller

這時會有兩種模組可以選擇

* Resource Controller（7 個方法）、通常放在 web.php
1. index() – 顯示資源列表
2. create() – 顯示建立新資源的表單
3. store() – 儲存新建立的資源
4. show($id) – 顯示單一資源內容
5. edit($id) – 顯示編輯資源的表單
6. update($id) – 更新既有的資源
7. destroy($id) – 刪除資源
* API resource controller（5 個方法）、通常放在 api.php中(需要另外建立)
1. index() – 列表
2. store() – 新增
3. show($id) – 查看
4. update($id) – 更新
5. destroy($id) – 刪除

分別在既有的maker:controller之後加上 --resource或是--api
```
php artisan make:controller UserController // 最陽春的指令
php artisan make:controller ExerciseTypeController --resource //esource Controller版
php artisan make:controller ExerciseRecordController --api //API resource controller
```
我這邊是使用API resource，所以之後會在routes新建一個api.php的文件

# API路由建立

Laravel 的 routes/web.php 和 routes/api.php 都是用來定義路由，但設計上有幾個重要的差別：

簡單的說
web.php = 傳統網站（有 session、有 CSRF、瀏覽器導向）
api.php = API（stateless、JSON 回應、給前端或手機用）

基本上在controller建立的方法必須逐一在api.php上做設定
```
Route::get('/users', [UserController::class, 'index']);
Route::get('/users/{id}', [UserController::class, 'show']);
Route::post('/users', [UserController::class, 'store']);
Route::put('/users/{id}', [UserController::class, 'update']);
Route::delete('/users/{id}', [UserController::class, 'destroy']);
```
但是如果只有API resource controller預設的那五個方法，就可以直接用Route::apiResource來處理
```
Route::apiResource('users', UserController::class);
```

我的api.php就會像這樣
```
<?php

use App\Http\Controllers\UserController;
use App\Http\Controllers\ExerciseTypeController;
use App\Http\Controllers\ExerciseRecordController;
use Illuminate\Support\Facades\Route;

Route::apiResource('users', UserController::class);
Route::apiResource('exercise-types', ExerciseTypeController::class);
Route::apiResource('exercise-records', ExerciseRecordController::class);
```

# 將api.php掛進路由系統
最後在bootstrap/app.php中加上 
```
api: __DIR__.'/../routes/api.php',
```
把 api.php 這個檔案掛進整個路由系統
bootstrap/app.php
```
<?php

use App\Http\Middleware\HandleAppearance;
use App\Http\Middleware\HandleInertiaRequests;
use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;
use Illuminate\Http\Middleware\AddLinkHeadersForPreloadedAssets;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        api: __DIR__.'/../routes/api.php',//加上這行
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        $middleware->encryptCookies(except: ['appearance', 'sidebar_state']);

        $middleware->web(append: [
            HandleAppearance::class,
            HandleInertiaRequests::class,
            AddLinkHeadersForPreloadedAssets::class,
        ]);
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })->create();
```

# 驗證

這實在終端機上執行指令
```
php artisan route:list --path=api
```
就會顯示出剛剛設定的15個api路由
```
GET|HEAD        api/exercise-records .......................... exercise-records.index › ExerciseRecordController@index
  POST            api/exercise-records .......................... exercise-records.store › ExerciseRecordController@store
  GET|HEAD        api/exercise-records/{exercise_record} .......... exercise-records.show › ExerciseRecordController@show
  PUT|PATCH       api/exercise-records/{exercise_record} ...... exercise-records.update › ExerciseRecordController@update
  DELETE          api/exercise-records/{exercise_record} .... exercise-records.destroy › ExerciseRecordController@destroy
  GET|HEAD        api/exercise-types ................................ exercise-types.index › ExerciseTypeController@index
  POST            api/exercise-types ................................ exercise-types.store › ExerciseTypeController@store
  GET|HEAD        api/exercise-types/{exercise_type} .................. exercise-types.show › ExerciseTypeController@show
  PUT|PATCH       api/exercise-types/{exercise_type} .............. exercise-types.update › ExerciseTypeController@update
  DELETE          api/exercise-types/{exercise_type} ............ exercise-types.destroy › ExerciseTypeController@destroy
  GET|HEAD        api/users .......................................................... users.index › UserController@index
  POST            api/users .......................................................... users.store › UserController@store
  GET|HEAD        api/users/{user} ..................................................... users.show › UserController@show
  PUT|PATCH       api/users/{user} ................................................. users.update › UserController@update
  DELETE          api/users/{user} ............................................... users.destroy › UserController@destroy

                                                                                                      Showing [15] routes
```

這樣就代表你的api架設成功，可以啟動server，用postman做測試
```
php artisan serve //啟動伺服器
```