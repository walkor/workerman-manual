# 基本流程
（以一個簡單的Websocket聊天室服務端為例）

#### 1、任意位置建立專案目錄
如 SimpleChat/
進入目錄執行 `composer require workerman/workerman`

#### 2、引入`vendor/autoload.php` (composer安裝後生成)
創建 start.php ，引入`vendor/autoload.php` 
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';
```

#### 3、選定協議
這裡我們選定Text文本協議(Workerman中自定義的一個協議，格式為文本+換行)

（目前Workerman支持HTTP、Websocket、Text文本協議，如果需要使用其他協議，請參照協議一章開發自己的協議）

#### 4、根據需要寫入口起動腳本
例如下面這個是一個簡單的聊天室的入口檔案。

SimpleChat/start.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$global_uid = 0;

// 當客戶端連上來時分配uid，並保存連接，並通知所有客戶端
function handle_connection($connection)
{
    global $text_worker, $global_uid;
    // 為這個連接分配一個uid
    $connection->uid = ++$global_uid;
}

// 當客戶端發送消息過來時，轉發給所有人
function handle_message(TcpConnection $connection, $data)
{
    global $text_worker;
    foreach($text_worker->connections as $conn)
    {
        $conn->send("user[{$connection->uid}] said: $data");
    }
}

// 當客戶端斷開時，廣播給所有客戶端
function handle_close($connection)
{
    global $text_worker;
    foreach($text_worker->connections as $conn)
    {
        $conn->send("user[{$connection->uid}] logout");
    }
}

// 創建一個文本協議的Worker監聽2347介面
$text_worker = new Worker("text://0.0.0.0:2347");

// 只啟動1個進程，這樣方便客戶端之間傳輸數據
$text_worker->count = 1;

$text_worker->onConnect = 'handle_connection';
$text_worker->onMessage = 'handle_message';
$text_worker->onClose = 'handle_close';

Worker::runAll();

```

#### 5、測試
Text協議可以用telnet命令測試
```shell
telnet 127.0.0.1 2347
```
