# 範例1
**``` (要求Workerman版本>=3.3.0) ```**

基於Worker的多進程(分佈式集群)推送系統，集群群發、集群廣播。

`start_channel.php`
整個系統只能部署一個start_channel服務。假設運行在192.168.1.1。
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// 初始化一個Channel服務端
$channel_server = new Channel\Server('0.0.0.0', 2206);

Worker::runAll();
```

`start_ws.php`
整個系統可以部署多個start_ws服務，假設運行在 192.168.1.2和192.168.1.3 兩台伺服器上。
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// websocket服務端
$worker = new Worker('websocket://0.0.0.0:4236');
$worker->count=2;
$worker->name = 'pusher';
$worker->onWorkerStart = function($worker)
{
    // Channel客戶端連接到Channel服務端
    Channel\Client::connect('192.168.1.1', 2206);
    // 以自己的進程id為事件名稱
    $event_name = $worker->id;
    // 訂閱worker->id事件並註冊事件處理函數
    Channel\Client::on($event_name, function($event_data)use($worker){
        $to_connection_id = $event_data['to_connection_id'];
        $message = $event_data['content'];
        if(!isset($worker->connections[$to_connection_id]))
        {
            echo "connection not exists\n";
            return;
        }
        $to_connection = $worker->connections[$to_connection_id];
        $to_connection->send($message);
    });

    // 訂閱廣播事件
    $event_name = '廣播';
    // 收到廣播事件後向當前進程內所有客戶端連接發送廣播數據
    Channel\Client::on($event_name, function($event_data)use($worker){
        $message = $event_data['content'];
        foreach($worker->connections as $connection)
        {
            $connection->send($message);
        }
    });
};

$worker->onConnect = function(TcpConnection $connection)use($worker)
{
    $msg = "workerID:{$worker->id} connectionID:{$connection->id} connected\n";
    echo $msg;
    $connection->send($msg);
};
Worker::runAll();
```

`start_http.php`
整個系統可以部署多個start_ws服務，假設運行在 192.168.1.4和192.168.1.5 兩台伺服器上。
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 用來處理http請求，向任意客戶端推送數據，需要傳workerID和connectionID
$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->name = 'publisher';
$http_worker->onWorkerStart = function()
{
    Channel\Client::connect('192.168.1.1', 2206);
};
$http_worker->onMessage = function(TcpConnection $connection, $request)
{
    // 兼容workerman4.x
    if (!is_array($request)) {
            $_GET = $request->get();
   }
    $connection->send('ok');
    if(empty($_GET['content'])) return;
    // 是向某個worker進程中某個連接推送數據
    if(isset($_GET['to_worker_id']) && isset($_GET['to_connection_id']))
    {
        $event_name = $_GET['to_worker_id'];
        $to_connection_id = $_GET['to_connection_id'];
        $content = $_GET['content'];
        Channel\Client::publish($event_name, array(
           'to_connection_id' => $to_connection_id,
           'content'          => $content
        ));
    }
    // 是全局廣播數據
    else
    {
        $event_name = '廣播';
        $content = $_GET['content'];
        Channel\Client::publish($event_name, array(
           'content'          => $content
        ));
    }
};

Worker::runAll();
```

## 測試 
1、運行個個伺服器上的服務

2、客戶端連接伺服器

打開chrome瀏覽器，按F12打開調試控制台，在Console一欄輸入(或者把下面代碼放入到html頁面用js運行)

```javascript
// 也可以連ws://192.168.1.3:4236
ws = new WebSocket("ws://192.168.1.2:4236");
ws.onmessage = function(e) {
    alert("收到服務端的消息：" + e.data);
};
```

3、通過調用http接口推送

url訪問 ```http://192.168.1.4:4237/?content={$content}```  或者  ```http://192.168.1.5:4237/?content={$content}```向所有客戶端連接推送```$content```數據

url訪問 ```http://192.168.1.4:4237/?to_worker_id={$worker_id}&to_connection_id={$connection_id}&content={$content}``` 或者```http://192.168.1.5:4237/?to_worker_id={$worker_id}&to_connection_id={$connection_id}&content={$content}``` 向某個worker進程中的某個客戶端連接推送```$content```數據

注意：測試時把 ```{$worker_id}``` ```{$connection_id}``` 和```{$content}``` 換成實際值
