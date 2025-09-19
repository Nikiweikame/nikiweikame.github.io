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
```
php artisan make:controller UserController
php artisan make:controller ExerciseTypeController
php artisan make:controller ExerciseRecordController
```