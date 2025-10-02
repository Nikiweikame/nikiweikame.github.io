---
title: 請AI協助我開發-part12
date: 2025-09-25 18:25:21
tags:  [AI , back-end]
---

## 其他元件的controller

要在controller文件中加入 jwt.auth 中間件
```
    // 在建構子中加上 jwt.auth 中間件
    public function __construct()
    {
        $this->middleware('jwt.auth');
    }
```
這樣就可以在後續的所有函式都會先進行token驗證，也能在後續的還是中使用
```
$user = auth()->user();
```
這種做法，配合api.php的運用會是這樣
```
Route::apiResource('exerciseTypes', ExerciseTypeController::class);
```

這部分會有兩個疑惑
其一，如果不是所有的方法都需要進行token驗證呢？
其二，jwt.auth 中間件能不能移到api.php統一處理？

針對其一的問題
如果你只想部分方法需要驗證，可以這樣寫：
假設CRUD中R的部分不需要token驗證，可以這樣呈現
```
public function __construct()
{
    // 只有 CUD 需要驗證
    $this->middleware('jwt.auth')->only(['store', 'update', 'destroy']);
}
```
或是相反，先套用全部，再排除：
```
public function __construct()
{
    // 除了 index/show 不需要驗證，其它都要驗證
    $this->middleware('jwt.auth')->except(['index', 'show']);
}
```

再來其二的問題，在api.php寫middleware，並把那些需要token驗證方法包進裡面

Route::get('/exerciseRecords', [ExerciseRecordController::class, 'index']);

Route::middleware('jwt.auth')->group(function () {
    // exercise records
    Route::post('/exerciseRecords', [ExerciseRecordController::class, 'store']);
    Route::put('/exerciseRecords/{id}', [ExerciseRecordController::class, 'update']);
    Route::delete('/exerciseRecords/{id}', [ExerciseRecordController::class, 'destroy']);
});


## ExerciseRecordController

## Read 
先功能面思考，我進到個人紀錄的時候，希望能看到我自己的運動紀錄
所以一開始要先從JWT取得user資料，
再從table中找到該user的資料，
之後將多筆資料透過get()輸出出去

```
        $user = auth()->user();
        $query = ExerciseRecord::where('user_id', $user->id);

        $records = $query->get();

        return response()->json([
            'success' => true,
            'data' => $records,
            'message' => 'Records fetched successfully',
        ]);
```
這時我思考著，會不會有使用者想看個別運動的記錄呢，就像是原本show功能的延伸
從查詢單一資料延伸成查詢單一運動項目的紀錄
```
    public function getRecords(Request $request)
    {
        $user = auth()->user(); // 從 JWT 拿使用者
        $exerciseTypeId = $request->input('exercise_type_id'); // 從前端拿條件

        $query = ExerciseRecord::where('user_id', $user->id);

        if ($exerciseTypeId) {
            $query->where('exercise_type_id', $exerciseTypeId);
        }

        $records = $query->get(); // 取得結果，多筆查詢

        return response()->json([
            'success' => true,
            'data' => $records,
            'message' => 'Records fetched successfully',
        ]);
    }
```
愛醬建議我這兩個功能可以整併在一起
在沒有給予Request時，回傳使用者所有紀錄
有Request時，則回傳使用者的特定運動紀錄
```
    public function index(Request $request)
    {
        $user = auth()->user();
        $exerciseTypeId = $request->input('exercise_type_id');

        $query = ExerciseRecord::where('user_id', $user->id);
        if ($exerciseTypeId) {
            $query->where('exercise_type_id', $exerciseTypeId);
        }

        $records = $query->get();

        return response()->json([
            'success' => true,
            'data' => $records,
            'message' => 'Records fetched successfully',
        ]);
    }
```

## Create

接著使用者新增運動紀錄store

一樣先從JWT獲得使用者資訊，接著驗證新增的資訊
再來是新增到資料庫上，最後將結果回傳出去
```

    public function store(Request $request)
    {
        // 取得當前使用者
        $user = auth()->user();
        // 驗證輸入
        $validated = $request->validate([
            'exercise_type_id' => 'required|exists:exercise_types,id',
            'record_time' => 'required|date',
            'count' => 'required|integer|min:1',
            'unit' => 'required|string|max:100',
            'calories' => 'required|numeric|min:0',
        ]);
        // 建立紀錄
        $record = ExerciseRecord::create([
            'user_id' => $user->id,
            'exercise_type_id' => $validated['exercise_type_id'],
            'record_time' => $validated['record_time'],
            'count' => $validated['count'],
            'unit' => $validated['unit'],
            'calories' => $validated['calories'],
        ]);
        // 回傳結果
        return response()->json([
            'success' => true,
            'data' => $record,
            'message' => 'Record created successfully',
        ], 201); // 201 Created
    }
```

## Update

有些人會先記錄自己預計要做的運動，但是計劃總是趕不上變化，
所以才需要修改功能

除了建立資料時的個欄位驗證之外，還需要精確地抓到腰修改的是哪一筆資料
所以輸入的參數update的fuction的參數除了Request之外還有放在api網址上的該筆紀錄的$id
```
    public function update(Request $request,$id)
```

而在查找資料的時候還必須加一層保護機制，確保該筆資料確實時這名使用者新增的，若該資料不存在則要回傳錯誤
```
        $record = ExerciseRecord::where('user_id', $user->id)
            ->where('id', $validated['id'])
            ->first();

        if (! $record) {
            return response()->json(['success' => false, 'message' => 'Record not found'], 404);
        }
```
之後將資料更新功能整合就會變成下面這樣
```

    public function update(Request $request,$id)
    {
        $user = auth()->user();

        $validated = $request->validate([
            'exercise_type_id' => 'required|exists:exercise_types,id',
            'record_time' => 'required|date',
            'count' => 'required|integer|min:1',
            'unit' => 'required|string|max:100',
            'calories' => 'required|numeric|min:0',
        ]);

        $record = ExerciseRecord::where('user_id', $user->id)
            ->where('id', $id)
            ->first();

        if (! $record) {
            return response()->json(['success' => false, 'message' => 'Record not found'], 404);
        }

        $record->update([
            'exercise_type_id' => $validated['exercise_type_id'],
            'record_time' => $validated['record_time'],
            'count' => $validated['count'],
            'unit' => $validated['unit'],
            'calories' => $validated['calories'],
        ]);

        return response()->json([
            'success' => true,
            'data' => $record,
            'message' => 'Record updated successfully',
        ], 200);
    }
```

## Delete

最後是CRUD的Delete刪除功能

跟update一樣必須精準地找到那筆資料，之後對那筆資料執行刪除動作
```

    public function destroy($id)
    {
        $user = auth()->user();
        $record = ExerciseRecord::where('user_id', $user->id)
            ->where('id', $id)
            ->first();

        if (! $record) {
            return response()->json([
                'success' => false,
                'message' => 'Record not found',
            ], 404);
        }

        $record->delete();

        return response()->json([
            'success' => true,
            'data' => null,
            'message' => 'Exercise record deleted successfully',
        ], 200);
    }
```
這樣基本的四個功能就有了，最後要在routes/api.php上，在JWT驗證路由中把exercise records的項目加上去就好

```
Route::post('/login', [AuthController::class, 'login']);
Route::post('/register', [AuthController::class, 'register']);

// 需要 JWT 驗證的路由
Route::middleware('jwt.auth')->group(function () {
    Route::get('/me', [AuthController::class, 'me']);
    Route::post('/logout', [AuthController::class, 'logout']);
    Route::put('/profile', [AuthController::class, 'updateProfile']);
    // exercise records
    Route::get('/exerciseRecords', [ExerciseRecordController::class, 'index']);
    Route::post('/exerciseRecords', [ExerciseRecordController::class, 'store']);
    Route::put('/exerciseRecords/{id}', [ExerciseRecordController::class, 'update']);
    Route::delete('/exerciseRecords/{id}', [ExerciseRecordController::class, 'destroy']);
});
```

