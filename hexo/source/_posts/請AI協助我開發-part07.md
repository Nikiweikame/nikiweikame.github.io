---
title: 請AI協助我開發-part07
date: 2025-09-04 15:19:53
tags: [diary]
---

## SQL 建立限制(SQL Create Constraints)

當我在做資料庫規劃的時候，想要在id做pk但是又擔心帳號欄位會有重複的問題，

在與人討論之後得知可以在資料表建立時的設定欄位屬性設定，就可以完美的解決問題

趁此來深入探討這部分的相關知識

## SQL Constraints 欄位限制

一個資料表除了資料表名稱與欄位名稱、資料型別之外，其實還有一個不容易會注意到的屬性
該屬性被稱為constraint，也就是對欄位資料的限制

像是最常見的pk(PRIMARY KEY)就是其中一種constraint

常見的constraint有以下幾種

* NOT NULL(NN) - Ensures that a column cannot have a NULL value
不能為空值
* UNIQUE(UQ) - Ensures that all values in a column are different
唯一值不能重複
* PRIMARY KEY(PK) - A combination of a NOT NULL and UNIQUE. Uniquely identifies each row in a table
pk唯一的識別欄位
* FOREIGN KEY - Prevents actions that would destroy links between tables
外鍵，主要是規範table間的關係，確保table的聯繫沒有問題
* CHECK - Ensures that the values in a column satisfies a specific condition
檢查，在資料填入欄位前，依設定的內容進行值得查核跟ENUM有點像但是彈性更大
* DEFAULT - Sets a default value for a column if no value is specified
預設值，在建立時若是沒有值時，會依預設得設定填入指定的值
* CREATE INDEX - Used to create and retrieve data from the database very quickly
索引，可以增加查找資料的速度
* AUTO INCREMENT - Auto-increment allows a unique number to be generated automatically when a new record is inserted into a table.
自動增加編號，同常運用在編號欄位的pk，確保pk的唯一不重複







