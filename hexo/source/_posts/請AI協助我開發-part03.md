---
title: 請AI協助我開發-part03
date: 2025-08-26 11:07:34
tags: [AI , back-end]
---

## 減肥紀錄網站資料庫-part03-運動項目與卡路里計算

接著要構思第三個table，運動項目與卡路里的計算
其實這部分的內容我有想寫在前端頁面中，但是這樣可擴展性就會比較差一點
而且運動減肥的項目千百種，如果不設計能動態資訊，這樣資料永遠都更新不完

接著是欄位的設計
pk先暫定為運動項目名稱
接著是該運動的強度(可null)，
還有運動計量單位，像是時間、距離或是次數
最後是消耗的卡路里，為了管理記錄還要有修改人以及修改日期的欄位
但是這就會出現另一個問題，運動項目名稱會重複
就表示這不適合作pk，pk必須選擇其他項目或是獨立的no

把我的規劃拿去問問AI，並得到肯定的回覆，並為我建立了table創建的語法

* Table 設計建議：exercise_type
```
CREATE TABLE exercise_type (
    id INT AUTO_INCREMENT PRIMARY KEY,       -- 流水號，PK
    name VARCHAR(50) NOT NULL,               -- 運動名稱 (跑步、深蹲、游泳…)
    intensity ENUM('light','medium','high') NULL, -- 強度，可為空
    unit ENUM('time','distance','count') NOT NULL, -- 計量單位
    calories_per_unit DECIMAL(10,2) NOT NULL, -- 每單位消耗卡路里
    updated_by VARCHAR(50) NOT NULL,         -- 修改人
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
                                            ON UPDATE CURRENT_TIMESTAMP
);
```

而且updated_at的欄位設計得比我想的還要完整，
如果沒有特別指定 updated_at，系統會自動填入「當下的時間」
當這筆資料被更新時（例如修改了運動名稱或卡路里數值），系統會自動把 updated_at 改成「當下的時間」。
這樣就不用額外去寫日期

此外他還建議我要不要再多一個欄位是建立時間
```
CREATE TABLE exercise_types (
    id INT AUTO_INCREMENT PRIMARY KEY,           -- 運動項目代號
    name VARCHAR(100) NOT NULL,                  -- 運動名稱
    intensity ENUM('light','medium','high') NULL, -- 強度，可為空
    unit ENUM('time', 'distance', 'count') NOT NULL, -- 計量單位
    calories_per_unit DECIMAL(6,2) NOT NULL,     -- 每單位消耗的卡路里
    updated_by VARCHAR(50) NOT NULL,             -- 修改人
    
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,   -- 建立時間
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP 
                                    ON UPDATE CURRENT_TIMESTAMP -- 最後修改時間
);
```

感覺要加一個描述，關於該運動項目的說明像是強度標準之類的，
不然無法客觀判定這項運動的強度為何？會增加使用者的誤判狀況
像是跑步的速率標準就需要客觀的依據，來區分消耗卡路里的量


```
CREATE TABLE exercise_types (
    id INT AUTO_INCREMENT PRIMARY KEY,           -- 運動項目代號
    name VARCHAR(100) NOT NULL,                  -- 運動名稱
    intensity ENUM('light','medium','high') NULL, -- 強度，可為空
    unit ENUM('time', 'distance', 'count') NOT NULL, -- 計量單位
    calories_per_unit DECIMAL(6,2) NOT NULL,     -- 每單位消耗的卡路里
    description TEXT NULL,                  -- 運動描述或強度標準
    updated_by VARCHAR(50) NOT NULL,             -- 修改人
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,   -- 建立時間
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP 
                                    ON UPDATE CURRENT_TIMESTAMP -- 最後修改時間
);
```

目前還不考慮將體重納入計算內，感覺會讓計算變得很複雜
現階段先把基本的功能實現，後續如果要補強在逐一處理

這樣三個table規劃完成了，之後就是API的規劃與應用