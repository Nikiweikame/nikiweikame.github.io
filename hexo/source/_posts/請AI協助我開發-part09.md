---
title: 請AI協助我開發-part09
date: 2025-09-09 10:44:08
tags: [AI , back-end]
---

# 建立database.sqlite

設定好 Migrations檔案之後，就要開始建立database.sqlite的內容了
```
//終端機
php artisan migrate
```

這邊可能會遇上的問題

```
//終端機回應
2025_08_28_030252_create_exercise_records_table ............................ 2.85ms DONE
2025_08_28_030252_create_exercise_types_table .............................. 0.48ms DONE
```

仔細看終端機的回覆可以發現，只有建立兩個table的訊息

我用 sqlite3 檢查 database.sqlite內的table，發現user table是有建立的，但是內容是laravel建立的預設內容
還有一些不認識的table，想必這些都是原本預設建立的table，此時回去看Migrations資料夾內的檔案，那些不認識的table name跟都對上了

這時候愛醬建議我下達php artisan migrate:refresh指令，
問題來了，因為我的migrate檔案有動過一些該drop的table沒寫進指令裡面，會讓這個操作不完整甚至會出現錯誤
正確的流程這可能要對migrate進行修改前，先把預設的table給drop掉才是
現在因為是剛開發階段，可以直接把資料砍掉重建

```
rm database/database.sqlite
touch database/database.sqlite
php artisan migrate
```
這樣我的database.sqlite也算是建立好了

# 建立model

之前在part05有提到一般開發的步驟（先 migration 後 model），雖然我忘記先去處理seeder的事務，
不意外的出了很多bug，才回來探討是哪邊出問題，原來是我漏了model這一步

之前也說了model是操作資料庫的入口，會用來寫 查詢、關聯、規則 等邏輯，其實不精確。
modelModel 是資料表的 PHP 代表，它負責：
* 跟資料表對應
* 讀取資料、寫入資料
* 定義欄位可批量寫入($fillable)
* 設定資料表關聯（hasMany、belongsTo…）
換句話說，Model 就是操作資料庫的「物件化橋樑」。

現在app/Model資料夾有三個檔案
此時app/Model就會有三個檔案
┌ ExerciseRecord.php
│ ExerciseType.php
└ User.php

這時候要分兩部分來學習，user跟user以外的
因為User Model 比普通 Model 多了 登入/認證功能。
學習User Model前我會建議先學習普通的model

## 一般的Model

```
<?php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class ExerciseType extends Model
{
    //
}
```
這是預設的model樣式，建立一個class並繼承Model(Illuminate\Database\Eloquent\Model)

* 允許建立假資料use HasFactory

use HasFactory; → 讓這個 Model 可以用 Factory 生成假資料

```
<?php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;
// Factory假資料
use Illuminate\Database\Eloquent\Factories\HasFactory;

class ExerciseType extends Model
{
    use HasFactory;
}
```

* $fillable（允許批量寫入的欄位）/$guarded（拒絕批量寫入的欄位）

這兩個是相對立的存在，通常建議使用$fillable 來明確控制安全欄位
確保用 create() 或 update() 批量寫入時，只有 $fillable 裡的欄位會被寫入
而$guarded 是跟 $fillable 反過來用，明確的拒絕批量寫入哪些欄位，通常都是以值會自動生成的欄位居多
再次強調：建議使用$fillable 來明確控制安全欄位
```
<?php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;
// Factory假資料
use Illuminate\Database\Eloquent\Factories\HasFactory;

class ExerciseType extends Model
{
    use HasFactory;

    protected $fillable = ['name', 'weight_unit', 'calories_per_unit', 'description', 'updated_by'];
}
```
* $table（自訂資料表名稱）
Laravel 預設會用 Model 名稱小寫加複數 當資料表名稱
例：User → users
如果資料表名稱不符合Laravel 預設規則，就要指定 $
來明確控制安全欄位
```
<?php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;
// Factory假資料
use Illuminate\Database\Eloquent\Factories\HasFactory;

class ExerciseType extends Model
{
    use HasFactory;

    protected $fillable = ['name', 'weight_unit', 'calories_per_unit', 'description', 'updated_by'];

    protected $table = 'exercise_types';
}
```

* $primaryKey（自訂主鍵欄位）
預設主鍵是 id
如果你的資料表主鍵不是 id，要設定 $primaryKey
```
protected $primaryKey = 'user_id';
```

* Timestamps 
Laravel 預設自動維護：
created_at
updated_at
如果資料表沒有這兩個欄位，可以關閉：
```
public $timestamps = false;
```

* 關聯關係（Relationships）

Eloquent 支援多種資料表間的關聯，如一對一、一對多、多對多等。
例如，一對多關聯：
```
public function posts()
{
    return $this->hasMany(Post::class);
} 
```
這就像ERD表示table間的關係，也是說明外鍵的連結
```
<?php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;
// Factory假資料
use Illuminate\Database\Eloquent\Factories\HasFactory;

class ExerciseType extends Model
{
    use HasFactory;

    protected $fillable = ['name', 'weight_unit', 'calories_per_unit', 'description', 'updated_by'];

    protected $table = 'exercise_types';

    public function exerciseRecords()
    {
      return $this->hasMany(ExerciseRecord::class);
    }
}
```

## user的Model

* $hidden 設定

user這是管控使用漲登入資訊的table，會有一些安全的機制

像是 不會被序列化的欄位 $hidden

這邊會規範哪些欄位不會被序列化，也就是無法轉換成JSON外傳
```
protected $hidden = ['password_hash'];
```


## 其他沒有提到的 Casts 型別轉換

確保某些欄位，在寫入資料的時候會自動轉換型別

```
protected function casts(): array
{
    return [
        'last_login_at' => 'datetime',
        'password_change_at' => 'datetime',
        'weight' => 'decimal:2',
    ];
}
```