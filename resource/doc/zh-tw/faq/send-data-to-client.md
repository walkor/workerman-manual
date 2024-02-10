# WorkerMan中如何向某個特定客戶端發送數據
使用worker來做伺服器，沒有使用GatewayWorker，該如何實現向指定用戶推送消息？

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 初始化一個worker容器，監聽1234端口
$worker = new Worker('websocket://workerman.net:1234');
// ====這裡進程數必須必須必須設置為1====
$worker->count = 1;
// 新增加一個屬性，用來保存uid到connection的映射(uid是用戶id或者客戶端唯一標識)
$worker->uidConnections = array();
// 當有客戶端發來消息時執行的回調函數
$worker->onMessage = function(TcpConnection $connection, $data)
{
    global $worker;
    // 判斷當前客戶端是否已經驗證,即是否設置了uid
    if(!isset($connection->uid))
    {
       // 沒驗證的話把第一個包當做uid（這裡為了方便演示，沒做真正的驗證）
       $connection->uid = $data;
       /* 保存uid到connection的映射，這樣可以方便的通過uid查找connection，
        * 實現針對特定uid推送數據
        */
       $worker->uidConnections[$connection->uid] = $connection;
       return $connection->send('login success, your uid is ' . $connection->uid);
    }
    // 其它邏輯，針對某個uid發送 或者 全局廣播
    // 假設消息格式為 uid:message 時是對 uid 發送 message
    // uid 為 all 時是全局廣播
    list($recv_uid, $message) = explode(':', $data);
    // 全局廣播
    if($recv_uid == 'all')
    {
        broadcast($message);
    }
    // 給特定uid發送
    else
    {
        sendMessageByUid($recv_uid, $message);
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
    }
}

// 運行所有的worker（其實當前只定義了一個）
Worker::runAll();
```
**說明：**
以上例子可以針對uid推送，雖然是單進程，但是支持10W在線是沒問題的。

注意這個例子只能單進程，也就是$worker->count 必須是1。要支持多進程或者伺服器集群的話需要Channel組件完成進程間通訊，開發也非常簡單，可以參考[Channel組件集群推送例子](../components/channel-examples.md)一節。

**如果希望在其他系統中推送消息給客戶端，可以參考[在其他項目中推送](push-in-other-project.md)一節**
