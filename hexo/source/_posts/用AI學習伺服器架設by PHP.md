---
title: 用AI學習伺服器架設by PHP-2025.08.07
date: 2025-08-07 13:47:39
tags: [PHP]
---

# 用AI學習伺服器架設by PHP

問：我想學伺服器架設by PHP 請你協助我

步驟 1：設定 PHP 開發環境

要用 PHP 架設伺服器，首先需要一個可以運行 PHP 的環境。以下是設定步驟：
1. 安裝 PHP
Mac：使用 Homebrew 安裝，執行 brew install php

實作：
安裝很順利沒有什麼問題

2. 安裝 Web 伺服器
PHP 通常需要搭配 Web 伺服器（如 Apache 或 Nginx）運行。以下是兩種常見選擇：

 * Apache：

下載並安裝 Apache（或使用 XAMPP/WAMP，這些是整合包，包含 Apache、PHP 和 MySQL）。
XAMPP 下載：XAMPP 官方網站。
安裝後，將 PHP 檔案放在 XAMPP 的 htdocs 資料夾。

* 內建 PHP 伺服器（適合開發測試）：

PHP 自帶一個簡單的開發用伺服器，無需額外安裝 Apache 或 Nginx。
在終端機進入你的 PHP 專案資料夾，執行：
bashphp -S localhost:8000

然後在瀏覽器輸入 http://localhost:8000 ，即可看到你的 PHP 頁面。

實作：
我這邊先選擇使用：內建 PHP 伺服器，
並透過bashphp -S localhost:8000建設PHP內建伺服器，但是目前我創建的PHP資料夾中沒有任何文件，所以無法顯示任何內容

3. 測試環境

在你的專案資料夾（例如 htdocs 或自訂資料夾）建立一個 index.php 檔案，內容如下：
<?php
phpinfo();
?>

開啟瀏覽器，輸入 http://localhost:8000 （或 Apache 的預設端口 80，如 http://localhost）。
如果看到 PHP 資訊頁面，代表環境設定成功！

實作：
我建立index.php 檔案，內容如上面所述，當我進入http://localhost:8000頁面時 ，可以看到PHP的資訊介面
表示我環境設定成功了。

步驟 2：用 PHP 撰寫簡單伺服器程式
以下是一個簡單的 PHP 伺服器範例，用於處理 HTTP 請求並回傳資料。
範例 1：基本 Hello World 伺服器

建立檔案 server.php：
<?php
// 設定回應標頭
header('Content-Type: text/plain');

// 回傳簡單訊息
echo "Hello, World! This is my PHP server.";
?>

使用 PHP 內建伺服器運行：
bashphp -S localhost:8000

在瀏覽器輸入 http://localhost:8000/server.php ，應該會顯示「Hello, World! This is my PHP server.」。

實作：
我建立server.php 檔案，內容如上面所述，當我進入http://localhost:8000server.php 頁面時，
確實顯示「Hello, World! This is my PHP server.」，所有程式最開始目標「Hello, World」成功完成。

範例 2：處理 GET 請求
這個範例會根據 URL 參數回傳客製化訊息：
<?php
header('Content-Type: text/plain');

// 取得 GET 參數
$name = isset($_GET['name']) ? $_GET['name'] : 'Guest';
echo "Hello, $name!";
?>

存為 greet.php，然後訪問 http://localhost:8000/greet.php?name=Alice ，會顯示「Hello, Alice!」。

實作：
建立greet.php 檔案，內容如上面所述，
我分別使用
http://localhost:8000/greet.php 與 http://localhost:8000/greet.php?name=Alice 各別訪問一次
分別得到「Hello, Guest!」與「Hello, Alice!」

範例 3：簡單 API
以下是一個簡單的 RESTful API，返回 JSON 格式資料：
php<?php
header('Content-Type: application/json');

// 模擬資料
$data = [
    'id' => 1,
    'name' => 'John Doe',
    'email' => 'john@example.com'
];

// 回傳 JSON
echo json_encode($data);
?>

存為 api.php，訪問 http://localhost:8000/api.php，會得到 JSON 格式的資料。


延伸：什麼是RESTful API ?

REST (Representational State Transfer)是一種軟體架設風格，
注意這是風格不是規範，他強調統一的介面，而他的架構約束被非官方稱為RESTful

正式的 REST 約束如下：

Client/Server – Clients are separated from servers by a well-defined interface
用戶端/伺服器 – 客戶端通過定義明確的介面與伺服器分開
Stateless – A specific client does not consume server storage when the client is "at rest"
無狀態 – 當客戶端處於「靜態」狀態時，特定用戶端不會消耗伺服器存儲
Cache – Responses indicate their own cacheability
緩存 – 回應指示它們自己的可緩存性
Uniform interface  統一的介面
Layered system – A client cannot ordinarily tell whether it is connected directly to the end server, or to an intermediary along the way
分層系統 – 用戶端通常無法判斷它是直接連接到終端伺服器，還是連接到中間人
Code on demand (optional) – Servers are able to temporarily extend or customize the functionality of a client by transferring logic to the client that can be executed within a standard virtual machine
按需編碼（可選）——伺服器能夠通過將邏輯傳輸到可在標準虛擬機中執行的用戶端來臨時擴展或自定義用戶端的功能


符合REST設計風格的Web API稱為RESTful API。它從以下三個方面資源進行定義：
直觀簡短的資源位址：URI，比如：http://example.com/resources。
傳輸的資源：Web服務接受與返回的網際網路媒體類型，比如：JSON，XML，YAML等。
對資源的操作：Web服務在該資源上所支援的一系列請求方法（比如：POST，GET，PUT或DELETE）。

RESTful API 需要請求包含以下主要元件：

唯一資源識別符
伺服器使用唯一資源識別符來識別每個資源。針對 REST 服務，伺服器通常使用統一資源定位器 (URL) 來執行資源識別。URL 指定資源的路徑。URL 類似於您在瀏覽器中輸入的網站地址，以瀏覽任何網頁。URL 也稱為請求端點，清晰地向伺服器指定用戶端需要什麼。

方法
開發人員通常使用超文字傳輸協定 (HTTP) 來實作 RESTful API。HTTP 方法告知伺服器需要對資源做什麼。以下是四種常見的 HTTP 方法：

GET

用戶端使用 GET 來存取位於伺服器上指定 URL 的資源。他們可以快取 GET 請求，並在 RESTful API 請求中傳送參數，以指示伺服器篩選資料後再傳送。

POST

用戶端使用 POST 向伺服器傳送資料。其包含請求中的資料呈現。多次傳送相同 POST 請求的連帶結果是，具有多次建立相同資源的。

PUT

用戶端使用 PUT 更新伺服器上的現有資源。與 POST 不同，在 RESTful Web 服務中多次傳送相同的 PUT 請求會得到相同的結果。

DELETE

用戶端使用 DELETE 請求來移除資源。DELETE 請求可變更伺服器狀態。但是，如果使用者沒有適當的身分驗證，則請求會失敗。

HTTP 標頭
請求標頭是用戶端與伺服器之間交換的中繼資料。例如，請求標頭會指示請求和回應的格式，提供有關請求狀態的資訊等。

資料

REST API 請求可能包含 POST、PUT 和其他 HTTP 方法成功運作的資料。

參數

RESTful API 請求可包含參數，為伺服器提供更多關於需要做什麼的詳細資訊。以下是一些不同類型的參數：

指定 URL 詳細資訊的路徑參數。
請求更多資源相關資訊的查詢參數。
快速驗證用戶端的 Cookie 參數。

實作：
因為
header('Content-Type: application/json');
的設置

傳回的資料會是JSON格式，


步驟 3：進階功能
當你熟悉基本伺服器功能後，可以嘗試以下進階應用：

處理 POST 請求：
<?php
header('Content-Type: application/json');

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $input = json_decode(file_get_contents('php://input'), true);
    $response = ['message' => 'Received', 'data' => $input];
    echo json_encode($response);
} else {
    echo json_encode(['error' => 'Only POST requests are allowed']);
}
?>

使用工具如 Postman 發送 POST 

實作：
這次是要依請求方法來操作API，
用postman方法用post將資料傳進API裡面，當method被檢測為post時，會將表示收到的訊息及傳入的資料包裝好回傳回來
但是用其他方法時，則會回傳錯誤，並提示你只能使用post做請求


路由系統：

使用簡單的 PHP 路由來模擬框架功能：
php<?php
header('Content-Type: text/plain');

$path = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);

switch ($path) {
    case '/':
        echo "Welcome to the homepage!";
        break;
    case '/about':
        echo "About us page.";
        break;
    default:
        http_response_code(404);
        echo "404 Not Found";
        break;
}
?>

訪問 http://localhost:8000/about 會顯示「About us page.」。