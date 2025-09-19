---
title: 請AI協助我開發-part08
date: 2025-09-08 17:00:00
tags: [AI , back-end]
---

## 延續part-05的內容-認識migration-續

延續上次說到的我在規劃好資料庫之後，透過下面三個laravel內建的指令，在migration資料夾建立了三個檔案(嚴格來說是兩個user是已經內建的)
```
//終端機
php artisan make:model User -m
php artisan make:model ExerciseType -m
php artisan make:model ExerciseRecord -m
```

```
<?php

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
        Schema::create('exercise_records', function (Blueprint $table) {
            $table->id();
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('exercise_records');
    }
};
```

先前有提到up()跟down()的用途
現在接著說他們實際上做了什麼

首先先創立一個table
```
Schema::create('table名稱','建立table欄位資料的細部動作')
```
之後在規範這個table裡面有那些欄位，欄位的型別與限制，也就是後面的function
```
function (Blueprint $table) {
            $table->id();
            $table->timestamps();
        };
```
一開始他先規範參數 $table 是一個 Blueprint 物件，
這樣函式內部就能使用Blueprint的method
再來就是兩個比較常看的method id()跟timestamps()
這兩個都是語法糖，把常用欄位的建立，封裝成一個方便的方法
id()其實是bigIncrements('id');
相當於SQL建立欄位時的
```
id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY
```
而timestamps()是建立created_at 與 updated_at兩個欄位且可以存放NULL值，
等價於：
```
$table->timestamp('created_at')->nullable();
$table->timestamp('updated_at')->nullable();
```
再來就是其他沒有在預設中的method

我依據我要建立table來看
```
Table exercise_record {
  id int [primary key, increment]
  user_id varchar(50) [ref: > user.id]
  exercise_type_id varchar(100) [ref: > exercise_type.id]
  record_time datetime
  count int
  unit varchar(100)
  calories decimal(10,2)
  created_at datetime
  updated_at datetime
}
```
目前有四種類別varchar、decimal、int、datetime

他們對應的method是string、decimal、integer、dateTime

他們method的第一個參數是欄位名稱，後面就是其他的限制

像是
user_id varchar(50)
就是
$table->string('user_id', 50);
後面的50就是長度限制50

轉換後會變成這樣，當然也可以請愛醬幫忙轉換
```
    public function up(): void
    {
        Schema::create('exercise_record', function (Blueprint $table) {
            $table->id(); // id INT AUTO_INCREMENT PRIMARY KEY
            $table->string('user_id', 50);
            $table->string('exercise_type_id', 100);
            $table->dateTime('record_time');
            $table->integer('count');
            $table->string('unit', 100);
            $table->decimal('calories', 10, 2);
            $table->timestamps(); // created_at, updated_at
        });
    }
```

之後外鍵的設定，這邊分為四個部分
$table->foreign('user_id')	將 exercise_record 的 user_id 欄位設定為外鍵
->references('id')	指定這個外鍵要對應到 users 表的 id 欄位
->on('users')	指定對應的資料表是 users
->onDelete('cascade')	刪除行為：當 users 表中對應的 id 被刪除時，exercise_record 裡對應的資料也會被自動刪掉（cascade 刪除）

整合起來就變這樣


```
    public function up(): void
    {
        Schema::create('exercise_record', function (Blueprint $table) {
            $table->id(); // id INT AUTO_INCREMENT PRIMARY KEY
            $table->string('user_id', 50);
            $table->string('exercise_type_id', 100);
            $table->dateTime('record_time');
            $table->integer('count');
            $table->string('unit', 100);
            $table->decimal('calories', 10, 2);
            $table->timestamps(); // created_at, updated_at

            // 外鍵約束
            $table->foreign('user_id')->references('id')->on('users')->onDelete('cascade');
            $table->foreign('exercise_type_id')->references('id')->on('exercise_types')->onDelete('cascade');
        });
    }
```

語法建立的時候愛醬有將三個表格都加上$table->timestamps()
但是我不是所有欄位都需要這兩個項目，
對此愛醬是這樣給予我建議的
```
對大多數應用程式來說，保留 $table->timestamps() 是很方便的
可以用來排序、查詢最新資料、做審計紀錄
若真的不需要時間追蹤，可以不加，migration 也會正常運作
```

簡單說就是雖然不是必要，但是建議加，或許會有意想不到的用處