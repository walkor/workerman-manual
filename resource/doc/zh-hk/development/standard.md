# 開發規範

## 應用程式目錄

應用程式目錄可以放置在任何位置。

## 入口檔案

和nginx+PHP-FPM下的PHP應用程式一樣，WorkerMan中的應用程式也需要一個入口檔案，入口檔案名稱沒有要求，而且這個入口檔案是以PHP Cli方式運行的。

入口檔案中是創建監聽進程相關的代碼，例如下面的基於Worker開發的代碼片段。

test.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 創建一個Worker監聽2345端口，使用http協議通訊
$http_worker = new Worker("http://0.0.0.0:2345");

// 啟動4個進程對外提供服務
$http_worker->count = 4;

// 接收到瀏覽器發送的數據時回覆hello world給瀏覽器
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    // 向瀏覽器發送hello world
    $connection->send('hello world');
};

Worker::runAll();
```


## WorkerMan中的代碼規範

1、類採用首字母大寫的駝峰式命名，類檔案名稱必須與檔案內部類名相同，以便自動加載。

例如：
```php
class UserInfo
{
...
```


2、使用命名空間，命名空間名字與目錄路徑對應，並以開發者的專案根目錄為基準。

例如專案MyApp/，類檔案MyApp/MyClass.php因為在專案根目錄，所以命名空間省略。類檔案MyApp/Protocols/MyProtocol.php因為MyProtocol.php在MyApp專案的Protocols目錄下，所以要加上命名空間 ```namespace Protocols;```。

```php
namespace Protocols;
class MyProtocol
{
....
```


3、普通函數及變數名採用小寫加下劃線方式，例如
```php
$connection_list = array();
function get_connection_list()
{
....
```


4、類成員及類的方法採用首字母小寫的駝峰形式，例如：
```php
public $connectionList;
public function getConnectionList();
```


5、函數及類的參數採用小寫加下劃線方式
```php
function get_connection_list($one_param, $tow_param)
{
....
```
