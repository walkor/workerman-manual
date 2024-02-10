# 監聽
```php
void Worker::listen(void)
```
用於實例化Worker後執行監聽。

該方法主要用於在Worker進程啟動後動態創建新的Worker實例，能夠實現同一個進程監聽多個端口，支持多種協議。需要注意的是用這種方法只是在當前進程增加監聽，並不會動態創建新的進程，也不會觸發onWorkerStart方法。

例如一個http Worker啟動後實例化一個websocket Worker，那麼這個進程即能通過http協議訪問，又能通過websocket協議訪問。由於websocket Worker和http Worker在同一個進程中，所以它們可以訪問共同的內存變量，共享所有socket連接。可以做到接收http請求，然後操作websocket客戶端完成向客戶端推送數據，類似的效果。

**注意：**

如果PHP版本<=7.0，則不支持在多個子進程中實例化相同端口的Worker。例如A進程創建了監聽2016端口的Worker，那麼B進程就不能再創建監聽2016端口的Worker，否則會報```Address already in use```錯誤。例如下面的代碼是```無法```運行的。

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
// 4個進程
$worker->count = 4;
// 每個進程啟動後在當前進程新增一個Worker監聽
$worker->onWorkerStart = function($worker)
{
    /**
     * 4個進程啟動的時候都創建2016端口的Worker
     * 當執行到worker->listen()時會報Address already in use錯誤
     * 如果worker->count=1則不會報錯
     */
    $inner_worker = new Worker('http://0.0.0.0:2016');
    $inner_worker->onMessage = 'on_message';
    // 執行監聽。這裡會報Address already in use錯誤
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

// 運行worker
Worker::runAll();
```

如果您的PHP版本>=7.0，可以設置Worker->reusePort=true， 這樣可以做到多個子進程創建相同端口的Worker。見下面的例子：
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2015');
// 4個進程
$worker->count = 4;
// 每個進程啟動後在當前進程新增一個Worker監聽
$worker->onWorkerStart = function($worker)
{
    $inner_worker = new Worker('http://0.0.0.0:2016');
    // 設置端口複用，可以創建監聽相同端口的Worker（需要PHP>=7.0）
    $inner_worker->reusePort = true;
    $inner_worker->onMessage = 'on_message';
    // 執行監聽。正常監聽不會報錯
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

// 運行worker
Worker::runAll();
```


### 範例 php後端及時推送訊息給客戶端

**原理：**

1、建立一個websocket Worker，用來維持客戶端長連接

2、websocket Worker內部建立一個text Worker

3、websocket Worker 與 text Worker是同一個進程，可以方便的共享客戶端連接

4、某個獨立的php後臺系統通過text協議與text Worker通訊

5、text Worker操作websocket連接完成數據推送

**代碼及步驟**

push.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 初始化一個worker容器，監聽1234端口
$worker = new Worker('websocket://0.0.0.0:1234');

/*
 * 注意這裡進程數必須設置為1
 */
$worker->count = 1;
// worker進程啟動後創建一個text Worker以便打開一個內部通訊端口
$worker->onWorkerStart = function($worker)
{
    // 開啟一個內部端口，方便內部系統推送數據，Text協議格式 文本+換行符
    $inner_text_worker = new Worker('text://0.0.0.0:5678');
    $inner_text_worker->onMessage = function(TcpConnection $connection, $buffer)
    {
        // $data數組格式，裡面有uid，表示向那個uid的頁面推送數據
        $data = json_decode($buffer, true);
        $uid = $data['uid'];
        // 通過workerman，向uid的頁面推送數據
        $ret = sendMessageByUid($uid, $buffer);
        // 返回推送結果
        $connection->send($ret ? 'ok' : 'fail');
    };
    // ## 執行監聽 ##
    $inner_text_worker->listen();
};
// 新增加一個屬性，用來保存uid到connection的映射
$worker->uidConnections = array();
// 當有客戶端發來訊息時執行的回調函數
$worker->onMessage = function(TcpConnection $connection, $data)
{
    global $worker;
    // 判斷當前客戶端是否已經驗證,既是否設置了uid
    if(!isset($connection->uid))
    {
       // 沒驗證的話把第一個包當做uid（這裡為了方便演示，沒做真正的驗證）
       $connection->uid = $data;
       /* 保存uid到connection的映射，這樣可以方便的通過uid查找connection，
        * 實現針對特定uid推送數據
        */
       $worker->uidConnections[$connection->uid] = $connection;
       return;
    }
};

// 當有客戶端連接斷開時
$worker->onClose = function(TcpConnection $connection)
{
    global $worker;
    if(isset($connection->uid))
    {
        // 連接斷開時刪除映射
        unset($worker->uidConnections[$connection->uid]);
    }
};

// 向所有驗證的用戶推送數據
function broadcast($message)
{
   global $worker;
   foreach($worker->uidConnections as $connection)
   {
        $connection->send($message);
   }
}

// 針對uid推送數據
function sendMessageByUid($uid, $message)
{
    global $worker;
    if(isset($worker->uidConnections[$uid]))
    {
        $connection = $worker->uidConnections[$uid];
        $connection->send($message);
        return true;
    }
    return false;
}

// 運行所有的worker
Worker::runAll();
```

啟動後端服務
 ```php push.php start -d```

前端接收推送的js代碼
```javascript
var ws = new WebSocket('ws://127.0.0.1:1234');
ws.onopen = function(){
    var uid = 'uid1';
    ws.send(uid);
};
ws.onmessage = function(e){
    alert(e.data);
};
```

後端推送訊息的代碼
```php
// 建立socket連接到內部推送端口
$client = stream_socket_client('tcp://127.0.0.1:5678', $errno, $errmsg, 1);
// 推送的數據，包含uid字段，表示是給這個uid推送
$data = array('uid'=>'uid1', 'percent'=>'88%');
// 發送數據，注意5678端口是Text協議的端口，Text協議需要在數據末尾加上換行符
fwrite($client, json_encode($data)."\n");
// 讀取推送結果
echo fread($client, 8192);
```
