# 開發規範

## 應用程式目錄

應用程式目錄可放置於任意位置。

## 入口檔案

與 nginx+PHP-FPM 下的 PHP 應用程式一樣，Workerman 中的應用程式也需要一個入口檔案，入口檔案名稱沒有特定要求，並且這個入口檔案是以 PHP Cli 方式運行。

入口檔案中包含建立監聽進程相關的程式碼，例如以下基於 Worker 開發的程式碼片段：

test.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 創建一個在 2345 端口監聽，使用 http 協議通訊的 Worker
$http_worker = new Worker("http://0.0.0.0:2345");

// 啟動 4 個進程以提供服務
$http_worker->count = 4;

// 當接收到瀏覽器發送的資料時，回覆 "hello world" 給瀏覽器
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    // 傳送 "hello world" 給瀏覽器
    $connection->send('hello world');
};

Worker::runAll();
```

## Workerman 中的程式碼規範

1. 類別使用大寫駝峰式命名，檔案名稱必須與內部類別名稱相同，以便自動載入。例如：
```php
class UserInfo
{
...
```

2. 使用命名空間，命名空間名稱與目錄路徑對應，並以開發者的專案根目錄為基準。

例如專案 MyAPP/，類別檔案 MyAPP/MyClass.php 因為在專案根目錄，所以命名空間省略。類別檔案 MyAPP/Protocols/MyProtocol.php 因為 MyProtocol.php 在 MyAPP 專案的 Protocols 目錄下，所以需加上命名空間 `namespace Protocols;`，如下：
```php
namespace Protocols;
class MyProtocol
{
...
```

3. 普通函數及變數名稱採用小寫加底線方式，例如
```php
$connection_list = array();
function get_connection_list()
{
...
```

4. 類別成員及類別方法採用小寫開頭的駝峰形式，例如：
```php
public $connectionList;
public function getConnectionList();
```

5. 函數及類別的參數採用小寫加底線方式
```php
function get_connection_list($one_param, $tow_param)
{
...
```
