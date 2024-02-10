# 如何廣播(群發)資料

## 範例（定時廣播）

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:2020');
// 這個例子中進程數必須為1
$worker->count = 1;
// 進程啟動時設置一個定時器，定時向所有客戶端連接發送數據
$worker->onWorkerStart = function($worker)
{
    // 定時，每10秒一次
    Timer::add(10, function()use($worker)
    {
        // 遍歷當前進程所有的客戶端連接，發送當前伺服器的時間
        foreach($worker->connections as $connection)
        {
            $connection->send(time());
        }
    });
};
// 執行worker
Worker::runAll();
```

## 範例（群聊）

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:2020');
// 這個例子中進程數必須為1
$worker->count = 1;
// 客戶端發來消息時，廣播給其他用戶
$worker->onMessage = function(TcpConnection $connection, $message)use($worker)
{
    foreach($worker->connections as $connection)
    {
        $connection->send($message);
    }
};
// 執行worker
Worker::runAll();
```

## 說明：
**單進程：**
以上例子只能**單進程**（```$worker->count=1```），因為多進程時多個客戶端可能連接到不同的進程中，進程間的客戶端是隔離的，無法直接通訊，也就是A進程中無法**直接**操作B進程的客戶端connection對象發送數據。(要做到這點，需要進程間通訊，比如可以使用Channel組件，例如[例子-集群發送](../components/channel-examples.md)、[例子-分組發送](../components/channel-examples2.md))。

**建議用GatewayWorker**
在workerman基礎上開發的GatewayWoker框架提供了更方便推送機制，包括組播、廣播等，可以設置多進程甚至可以多伺服器部署，如果需要給客戶端推送數據，建議使用GatewayWorker框架。

GatewayWorker手冊地址 https://doc2.workerman.net
