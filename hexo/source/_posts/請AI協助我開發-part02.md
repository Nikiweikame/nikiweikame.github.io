---
title: 請AI協助我開發-part02
date: 2025-08-21 13:44:18
tags: [diary]
---


## 減肥紀錄網站資料庫-part01-帳號管理

不管是透過線上的資料庫或是本機的機料庫，我都需要設計一個這個網站需要的資料庫資料，
如果可以的話，我希望能分三種模式，
第一種是本地建立的資料庫，也就是自記開發API來串接錢後端
第二種是運用線上資料庫，透過該網站提供的API來串接線上資料庫，達成資料皆換
第三種就是僅在本地使用，將內容儲存在瀏覽器的內部站純空間，不過資料儲存受瀏覽器的約約數比較沒有長久性

首先進入我的網頁，必須先註冊/登入才能建立/讀取個人的資料
就要有帳號管理的table
pk欄位就設定帳號
再來就是密碼、帳號狀態

安全規範設置
登入時要顯示出上次登入的日期，讓使用者查核是否有異常登入狀況，
再來要記錄密碼使用是否超過三個月，所以table上還要有上次密碼變更日期
如果超過三個月要提醒是否更換密碼或是沿用舊密碼

後來想到，要怎麼稱呼使用者，總不能用帳號來稱呼使用者，所以要在欄位加上暱稱的欄位

```
CREATE TABLE users (
    user_id VARCHAR(50) PRIMARY KEY, -- 帳號（主鍵）
    password_hash VARCHAR(255) NOT NULL, -- 加密後的密碼
    nickname VARCHAR(50), -- 暱稱
    status ENUM('active', 'inactive', 'locked') DEFAULT 'active', -- 帳號狀態
    last_login_at DATETIME, -- 上次登入時間
    password_changed_at DATETIME -- 上次密碼變更時間
);
```

## 減肥紀錄網站資料庫-part02-資料紀錄


再來就是核心的功能，要記錄使用者運動記錄以及消耗的卡路里
首先是pk這個就沒有特定的項目就用no來表示，讓系統自動填入
之後就是使用者id，以及記錄的時間要區分年月日年月日各一個欄位，後續篩選比較方便
這部分要問問AI通常會用什麼方式做記錄

AI回答
如果是設計資料庫的話，通常會這樣考量：

直接用一個 DATETIME 或 DATE 欄位

資料庫（例如 MySQL、PostgreSQL）原生支援 DATE / DATETIME 型別，可以同時儲存年月日（甚至時分秒）。

篩選的時候可以透過 SQL 函數抽取特定部分，例如：

SELECT * FROM record WHERE YEAR(record_time) = 2025;
SELECT * FROM record WHERE MONTH(record_time) = 8;


這樣比較直覺，程式處理也方便。

把年、月、日分開各一個欄位

好處：後續要做篩選、統計（例如月報表、年報表）可能會快一點，不需要額外用函數拆解日期。

壞處：資料冗長（要三個欄位）、維護比較麻煩，尤其跨日期計算（例如計算兩次運動相隔幾天）會很不方便。

👉 一般業界做法：

建議用一個 DATE / DATETIME 欄位就好，必要時再透過 SQL 或應用層把年月日拆出來。

如果真的很常用到「只看年月日各自的篩選」，可以額外建立 索引或是產生計算欄位（Generated Column）來加速查詢，而不是一開始就拆成三個欄位。

這個回答時墳具有建設，也彌補了我經驗上的不足，這樣欄位就只剩下no、使用者id跟日期

之後就是要記錄的運動項目，運動的次數或是時間，再來就是消耗的卡路里三個的欄位

拿著我規劃的結果問問AI看看還有什麼需要補足的地方

AI回答：

* 運動次數/時間的設計

「次數」和「時間」本質上是不同單位（整數次數 vs. 時間長度）。

如果都塞在一個欄位，未來處理會麻煩（例如統計「總時長」或「總次數」會混在一起）。

建議改成兩個欄位：

duration_minutes（運動時間，分鐘數）

reps（次數，若不適用則填 NULL）

* 日期欄位

如果只記錄「一天一筆」就用 DATE。

如果要記錄「一天多筆」（例如早上跑步，晚上游泳），建議用 DATETIME，這樣才能區分時間點。

運動次數/時間的設計，建議我把欄位分開，這樣我區分為時長，距離以及次數三項

```
CREATE TABLE exercise_record (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id VARCHAR(50) NOT NULL,
    exercise_type_id INT NOT NULL,   -- 運動種類（跑步、舉重、瑜伽…）
    record_time DATETIME NOT NULL,   -- 紀錄時間
    duration_minutes DECIMAL(10,2) NULL, -- 運動時間（分鐘）
    distance_km DECIMAL(10,2) NULL,      -- 運動距離（公里）
    count INT NULL,                        -- 次數（舉重、深蹲等）
    calories INT NOT NULL,                -- 消耗卡路里
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);
```

這樣兩個核心table就完成了，隨然中間出了點小問題
像是user_id欄位的屬性沒有一致，導致創建時出現型態不同 → MySQL 3780 錯誤