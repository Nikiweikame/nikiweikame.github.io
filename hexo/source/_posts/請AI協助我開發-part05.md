---
title: 請AI協助我開發-part05
date: 2025-08-27 16:23:33
tags: [AI , back-end]
---

## 從SQLite開始的API製作

如果要使用SQLite就必須先建立一個空的 SQLite 資料庫檔案。

```
//終端機
touch database/database.sqlite
```

然後在 .env 設定SQLite 資料庫檔案位置
```
DB_CONNECTION=sqlite
DB_DATABASE=database/database.sqlite
```

### Eloquent建立

設定完成之後就要開始對資料庫做串聯，首先先用Laravel 的 Artisan 指令「產生程式骨架」，
```
//終端機
php artisan make:model User -m
php artisan make:model ExerciseType -m
php artisan make:model ExerciseRecord -m
```

從Laravel文件中所描述
這是Laravel的物件關係映射器object-relational mapper (ORM)，可讓您愉快地與資料庫互動。
使用 Eloquent 時，每個資料庫表都有一個對應的「模型」，用於與該表互動。

換句話說就是可以透過這些model對這些table進行簡易的CRUD操作

* 注意：這只是先畫資料庫連結與操作的藍圖

model是在app/Model建立代表model的php檔案，是操作資料庫的入口，會用來寫 查詢、關聯、規則 等邏輯。

-m（等同 --migration）
Migration 是「資料表的版本控制腳本」，用來建立/修改資料表結構。
會在database/migrations/建立php文件
之後用 php artisan migrate 執行，資料庫才會真的產生/更新表。

Laravel 專案通常已內建 User Model 與 users 表，已有就不要重複建立

此時app/Model就會有三個檔案
┌ ExerciseRecord.php
│ ExerciseType.php
└ User.php

database/migrations/也會有三個檔案
┌ 0001_01_01_000000_create_users_table.php
│ xxxx_xx_xx_xxxxxx_create_exercise_records_table.php
└ xxxx_xx_xx_xxxxxx_create_exercise_types_table.php

而一般開發的步驟（先 migration 後 model）

### migrations設定

先針對ExerciseType的migrations做設定，考量User的有預設的關係會比較複雜(一次建立三個table)，
所以我選擇先對ExerciseType的migrations做設定，
```

<?php
//database/migrations/xxxx_xx_xx_xxxxxx_create_exercise_types_table.php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('exercise_types', function (Blueprint $table) {
            $table->id();
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('exercise_types');
    }
};
```
首先up()跟down()是個別對應
```
php artisan migrate
php artisan migrate:rollback
```
這兩個指令去新增資料表及刪除資料表，
對此我就有疑問，能不能針對個別檔案執行這些操作，愛醬對此問題是這樣回答的

答：
```
針對單一檔案的方法

Laravel 有提供 --path 與 --pretend 等參數，可以針對單一 migration 檔案操作：

1. 執行某一個 migration（建立）
php artisan migrate --path=/database/migrations/2025_08_28_123456_create_exercise_types_table.php


👉 這樣就只會執行這個檔案的 up()。

2. 回滾某一個 migration（刪除）

Laravel 沒有提供「直接 rollback 單檔」的指令，但有兩種方式：

方式 A：reset 後再指定 migrate

php artisan migrate:rollback --step=1


👉 如果你剛剛最後只跑過這個檔案，--step=1 就會只回滾這一個。

方式 B：用 migrate:refresh 搭配 --path

php artisan migrate:refresh --path=/database/migrations/2025_08_28_123456_create_exercise_types_table.php


👉 refresh 的意思是「先 down()，再 up()」，所以等於先刪掉再重建。
這也是你要針對單檔修改 schema 時最常用的做法。
一般狀況下
php artisan migrate
→ 執行 所有還沒執行過 的 migration（會跑每個檔案的 up()）。

php artisan migrate:rollback
→ 會依照「批次 (batch)」把最後一次 migrate 的 migration 全部 rollback（跑 down()）
```

看來只有建立可以做精細操作，刪除就沒辦法了
