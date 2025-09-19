---
title: 請AI協助我開發-part04
date: 2025-08-26 16:08:31
tags: [AI , back-end]
---

## PHP框架Laravel來進行API的開發-初始的困惑

目前我是使用Laravel預設的設置

所以我的.env檔案跟資料庫的連結是這樣

```
DB_CONNECTION=sqlite
# DB_HOST=127.0.0.1
# DB_PORT=3306
# DB_DATABASE=laravel
# DB_USERNAME=root
# DB_PASSWORD=
```

預設的DB使用的是SQLite，跟我原本預計使用的MySQL有點出入

我問了愛醬什麼是SQLite

答：
```
SQLite 可以想像成「一個檔案型的小型資料庫」，它跟 MySQL、PostgreSQL 這類資料庫不太一樣，特性如下：

🔹 1. 儲存方式

SQLite 不需要伺服器（Server）跑起來，資料都存在一個 .sqlite 或 .db 檔案裡。

例如：

database/database.sqlite

整個資料庫就是這一個檔案，所有表格、資料都在裡面。

🔹 2. 使用情境

適合小型專案、測試環境或單機應用。

不需要安裝 MySQL、PostgreSQL 伺服器。

Laravel 可以直接讀寫這個檔案，非常方便快速。

🔹 3. 與 MySQL、PostgreSQL 比較
特性	SQLite	MySQL / PostgreSQL
伺服器	不需要	需要伺服器
安裝	超簡單	需要安裝、設定
性能	小型專案很好	大型專案更穩定、效率高
多人同時寫入	弱（適合單機或少量使用者）	強（支援高併發）
檔案	單一檔案	多個資料表在資料庫伺服器中
🔹 4. Laravel 使用方式

在 .env 設定：

DB_CONNECTION=sqlite
DB_DATABASE=database/database.sqlite


不需要設定 host、username、password。

Laravel 就會直接把資料寫進那個檔案。

簡單比喻：

SQLite = Excel 檔案，每個 Sheet 是一個 Table；
MySQL/PostgreSQL = 大型資料庫伺服器，有多個使用者同時操作和管理權限。
```

這個比喻挺好懂的

SQLite = Excel 檔案

就像是本地的文件，類似excel儲存資料的檔案

我進一步問愛醬，SQLite與MySQL這兩個的差別，以及推薦使用哪個

答：

```
🔹 1️⃣ SQLite

適合情境：

專案規模小、使用者量不大。

主要用來 開發、測試或單機使用。

不想安裝 MySQL 伺服器。

優點：

安裝超簡單，直接用一個檔案就能跑。

Laravel 支援很好，快速建立 migration + seeder。

開發初期可以先用 SQLite，快速驗證 API 功能。

缺點：

多人同時寫入（高併發）會有限制。

大量資料或複雜查詢效能不如 MySQL。

🔹 2️⃣ MySQL（或 MariaDB）

適合情境：

專案可能會上線，預計有多位使用者同時操作。

資料量逐漸增加，需要穩定和高效能。

想要完整的使用者權限、備份、監控功能。

優點：

支援多人同時操作資料庫，高併發能力強。

大型專案和複雜查詢表現好。

容易與各種主機環境、雲端服務整合。

缺點：

需要安裝、設定伺服器（MySQL/MariaDB）。

開發初期比 SQLite 稍微麻煩。

🔹 我的建議

開發階段 → SQLite

快速跑起來測試功能、API。

migration、seeder 都可以直接用。

上線 / 部署階段 → MySQL

三張表不算多，但如果未來使用者增加或想做統計、報表，MySQL 更穩定。

💡 總結：

SQLite = 開發方便、快速、單機好用

MySQL = 上線穩定、多人同時使用、資料安全
```

針對他的建議
開發階段 → SQLite
上線 / 部署階段 → MySQL
我就有疑問，
業界會這樣做嗎？
還是直接就MySQL了

答：
```
1️⃣ 為什麼開發階段用 SQLite

簡單快速：SQLite 不需要啟動資料庫服務，只有一個檔案，開發環境可以直接用，快速測試。

方便測試 / CI：每個開發者可以有自己的 SQLite 檔案，互不影響。

零設定：不用管帳號、密碼、網路連線，尤其對小型專案或原型很方便。

Laravel 官方文檔也常用 SQLite 做開發測試（特別是單元測試）。

2️⃣ 為什麼部署 / 上線用 MySQL（或其他 Server DB）

多使用者 / 高併發：SQLite 的鎖定機制不適合多人同時寫入。

資料安全 / 備份：MySQL 支援更完整的權限管理、備份、replication。

擴充性：MySQL 可以水平擴展、加入讀寫分離、儲存大量資料。

3️⃣ 業界做法

小型專案 / 原型：開發階段用 SQLite，部署階段用 MySQL 或 PostgreSQL。

中大型專案：直接從開發就用 MySQL / PostgreSQL，確保開發和生產環境一致。

測試環境：有時候會用 SQLite 做單元測試，但實際開發也連線到 MySQL。
```

聽了愛醬的分析之後，我先從SQLite開始開發吧！之後再轉為MySQL，也為後續的測試鋪路

