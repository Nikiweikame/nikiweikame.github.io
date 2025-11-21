---
title: 從controller寫到DB
date: 2025-11-15 16:25:30
tags: [diary,back-end]
---

# 從controller寫到DB的常用的五層架構(Laravel為例)

感覺這部分需要從MVC開始說起
![MVC組件之間的典型合作-wiki百科](https://upload.wikimedia.org/wikipedia/commons/thumb/a/a0/MVC-Process.svg/1280px-MVC-Process.svg.png)
view是前端的畫面
model是屬於資料處理的部分，從商業邏輯到寫入DB
controller則是串接view與model的橋樑
view-controller-model
當專案變大時，我們會「在原本 MVC 基礎上再細分」
在controller-model間再往下細分
很多中型專案走到 Controller → Service → Model（Eloquent）三層
但如果專案更大更複雜會再往下延伸
Controller → Service → Repository → Model → DB
加上 FormRequest、DTO、Resource 處理輸入輸出

## Controller：流程、協調者

不放商業邏輯
不處理資料庫
只負責「流程」與「接收／回傳」

## Service：商業邏輯的核心

處理流程中的複雜邏輯
例如：
計算運動熱量
比對使用者權限
交易流程（建立訂單、扣庫存、寄通知）
Controller 只叫 Service，以保持乾淨

## Repository 的用途是：

專門處理 DB CRUD
讓 Service 不直接碰 ORM
將來換 DB（MySQL → MongoDB）也不用改 Service

小型～中大型專案：直接用 Eloquent/Model 就好
極大型／長壽命／有明確換 DB 需求（如同時支援 MySQL 跟 Elasticsearch）：才考慮 Repository 或 Data Mapper 模式

## Model：資料模型，有對資料直接訪問的權力

負責操作 DB
包含關聯（hasMany、belongsTo…）
不能塞太多邏輯，避免膨脹
小型～中型專案：直接在 Model 或 Service 裡用 Eloquent 就好
大型／長壽命／需要換 DB 時，才引入 Repository 模式

## DB（資料庫）

最後的儲存層