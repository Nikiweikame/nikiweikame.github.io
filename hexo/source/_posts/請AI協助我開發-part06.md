---
title: 請AI協助我開發-part06
date: 2025-08-28 16:45:31
tags:
---

## 嘗試製作這次系統設計文件

其實我也不確定這算是哪一種文件，我只是先把想的東西以及該有的東西做一次整理，

這樣也比較容易向他人展示我在做什麼，也方便做意見交流以及獲取建議

這次專案的主題叫做--好想做運動

1. 主要功能
* 帳號登入
* 使用者記錄運動項目與時間
* 依區間查閱使用者記錄的內容
* 新增運動項目與消耗的卡路里

2. 運用到的Table
* 帳號表（user）
────────────────────────────────────────────
欄位名稱            型別            備註
────────────────────────────────────────────
user_id            VARCHAR(50)     PK (帳號)
password_hash      VARCHAR(255)    密碼雜湊
nickname           VARCHAR(50)     暱稱
status             VARCHAR(50)     狀態 (active/inactive/locked)
last_login_at      DATETIME        上次登入時間
password_change_at DATETIME        上次更改密碼時間

* 運動紀錄表（exercise_record）
exercise_record
────────────────────────────────────────────
欄位名稱            型別            備註
────────────────────────────────────────────
id                 INT(AI)         PK (流水號)
user_id            VARCHAR(50)     使用者名稱
exercise_type      VARCHAR(100)    運動項目
record_time        DATETIME        紀錄時間
duration_minutes   DECIMAL(10,2)   運動持續時間(分鐘)
distance_km        DECIMAL(10,2)   距離(公里)
count              INT             次數
calories           INT             消耗卡路里

* 運動項目表（exercise_type）
exercise_type
────────────────────────────────────────────
欄位名稱            型別            備註
────────────────────────────────────────────
id                 INT(AI)         PK (流水號)
name               VARCHAR(100)    項目名稱 (e.g. 跑步)
intensity          VARCHAR(100)    強度 (high/medium/low)
unit               VARCHAR(100)    單位 (time/distance/count)
calories_per_unit  DECIMAL(6,2)    單位消耗卡路里
description        TEXT            說明
updated_by         VARCHAR(50)     修改者
created_at         DATETIME        建立時間
updated_at         DATETIME        修改時間

![ERD初版-土法煉鋼版](images/ERD_ver.init.png)

資料庫結構是這個專案很重要的核心部分，原本後續還有先嘗試著寫API文件
不過當我拿這份文件去跟其他人交流的時候，發現問題蠻多的API文件的部分要整個重寫

## 不要閉門造車，嘗試與人交流聆聽不同的意見

其實這上面的結果除了自己的思維之外，還透過跟愛醬的討論
但是在業界實際運用上，就有可能會有一些的出入
與不同人討論可以得到不同的觀點與收穫

### 得到的建議

* user的table也建立一個no流水號
* exercise_record與exercise_type相關聯的運動項目建議改為流水號的id
* exercise_record的欄位設計不夠直覺且有簡化的空間，duration_minutes、distance_km、count，可以簡化成數量及單位
* exercise_type的name與intensity可以整併為一個

這些建議反映了實務經驗與資料庫設計的最佳實踐，是我獨自研究無法輕易觸及的部分

user table 加流水號：用自增主鍵（如 id）有助於資料關聯、效能優化，也方便未來擴充。
exercise_record 與 exercise_type 關聯用 id：用 id 做關聯比用名稱更安全，避免名稱重複或修改造成資料錯誤。
欄位簡化：將 duration_minutes、distance_km、count 合併成「數量＋單位」能讓資料結構更彈性，方便支援更多運動類型。
name 與 intensity 整併：如果強度是運動名稱的一部分，合併可以簡化結構，但要評估未來是否會獨立查詢強度。

除此之外前輩們也給予我很多後端概念上的建議，對於前後段分離的實作上可以奠定更穩固的基礎
像是當一個計算前後端都能執行的時候，會建議給前端執行還是後端執行
* 跟「資料正確性 / 安全性」有關 → 一律放後端
* 只影響「體驗 / UI」的即時計算 → 放前端即可
* 複雜度高、效能吃重的 → 後端更合適

與有經驗的人交流可以獲得不少資訊，雖然我偏向I人，但是我還是覺得這部分多多跟人交流不會有壞處，不過之前寫的API全部都要做些修正了

## 新工具的得知與掌握

與人討論的過程中我也得知了一個新工具dbdiagram.io，
這可以幫你產出ERD

* 透過dbdiagram.io工具產生ERD

```
Table user {
  id int [primary key, increment]
  user_id varchar(50)
  password_hash varchar(255)
  nickname varchar(50)
  weight decimal(10,2)
  status varchar(50)
  last_login_at datetime
  password_change_at datetime
}

Table exercise_record {
  id int [primary key, increment]
  user_id varchar(50) [ref: > user.id]
  exercise_type_id varchar(100) [ref: > exercise_type.id]
  record_time datetime
  count int
  unit varchar(100)
  calories decimal(10,2)
  updated_at datetime
}

Table exercise_type {
  id int [primary key, increment]
  name varchar(100)
  weight_unit varchar(1)
  calories_per_unit decimal(6,2)
  description text
  updated_by varchar(50)
  created_at datetime
  updated_at datetime
}
```


![ERD修改版-dbdiagram版](images/ERD_ver.01.png)