＃ on
**```（要求Workerman版本>=3.3.0）```**
```php
void \Channel\Client::on(string $event_name, callback $callback_function)
```
訂閱```$event_name```事件並註冊事件發生時的回調```$callback_function```

## 回調函數的參數

 ``` $event_name ```

訂閱的事件名稱，可以是任意的字符串。

 ``` $callback_function ```

事件發生時觸發的回調函數。函數原型為```callback_function(mixed $event_data)```。```$event_data```是事件發布（publish）時傳遞的事件數據。

注意：

如果同一個事件註冊了兩個回調函數，後一個回調函數將覆蓋前一個回調函數。

## 范例
多進程Worker（多伺服器），一個客戶端發消息，廣播給所有客戶端

start.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 初始化一個Channel服務端
$channel_server = new Channel\Server('0.0.0.0', 2206);

// websocket服務端
$worker = new Worker('websocket://0.0.0.0:4236');
$worker->name = 'websocket';
$worker->count = 6;
// 每個worker進程啟動時
$worker->onWorkerStart = function($worker)
{
    // Channel客戶端連接到Channel服務端
    Channel\Client::connect('127.0.0.1', 2206);
    // 訂閱broadcast事件，並註冊事件回調
    Channel\Client::on('broadcast', function($event_data)use($worker){
        // 向當前worker進程的所有客戶端廣播消息
        foreach($worker->connections as $connection)
        {
            $connection->send($event_data);
        }
    });
};

$worker->onMessage = function(TcpConnection $connection, $data)
{
   // 將客戶端發來的數據當做事件數據
   $event_data = $data;
   // 向所有worker進程發布broadcast事件
   \Channel\Client::publish('broadcast', $event_data);
};

Worker::runAll();
```  

**測試**

打開chrome瀏覽器，按F12打開調試控制台，在Console一欄輸入(或者把下面代碼放入到html頁面用js運行)

接收消息的連接
```javascript
// 127.0.0.1換成實際workerman所在ip
ws = new WebSocket("ws://127.0.0.1:4236");
ws.onmessage = function(e) {
    alert("收到服務端的消息：" + e.data);
};
```

廣播消息
```
ws.send('hello world');
```
