# 如何實現異步任務

**問：**

如何異步處理繁重的業務，避免主業務被長時間阻塞。例如我要給1000用戶發送郵件，這個過程很慢，可能要阻塞數秒，這個過程中因為主流程被阻塞，會影響後續的請求，如何將這樣的繁重任務交給其他進程異步處理。

**答:**

可以在本機或者其他伺服器甚至伺服器集群預先建立一些任務進程處理繁重的業務，任務進程數可以開多一些，例如 CPU 的 10 倍，然後調用方利用 AsyncTcpConnection 將資料異步發送給這些任務進程異步處理，異步得到處理結果。

任務進程服務端
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// task worker，使用Text協議
$task_worker = new Worker('Text://0.0.0.0:12345');
// task進程數可以根據需要多開一些
$task_worker->count = 100;
$task_worker->name = 'TaskWorker';
$task_worker->onMessage = function(TcpConnection $connection, $task_data)
{
     // 假設發來的是 JSON 資料
     $task_data = json_decode($task_data, true);
     // 根據 task_data 處理相應的任務邏輯.... 得到結果，這裡省略....
     $task_result = ......
     // 發送結果
     $connection->send(json_encode($task_result));
};
Worker::runAll();
```

在 Workerman 中調用

```php
use Workerman\Worker;
use \Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// websocket 服務
$worker = new Worker('websocket://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $ws_connection, $message)
{
    // 與遠程 task 服務建立異步連接，IP 為遠程 task 服務的 IP，如果是本機就是 127.0.0.1，如果是集群就是 LVS 的 IP
    $task_connection = new AsyncTcpConnection('Text://127.0.0.1:12345');
    // 任務及參數資料
    $task_data = array(
        'function' => 'send_mail',
        'args'       => array('from'=>'xxx', 'to'=>'xxx', 'contents'=>'xxx'),
    );
    // 發送資料
    $task_connection->send(json_encode($task_data));
    // 異步獲得結果
    $task_connection->onMessage = function(AsyncTcpConnection $task_connection, $task_result)use($ws_connection)
    {
         // 結果
         var_dump($task_result);
         // 獲得結果後記得關閉異步連接
         $task_connection->close();
         // 通知對應的 websocket 客戶端任務完成
         $ws_connection->send('task complete');
    };
    // 執行異步連接
    $task_connection->connect();
};

Worker::runAll();
```

這樣，繁重的任務交給本機或者其他伺服器的進程去做，任務完成後會異步收到結果，業務進程就不會阻塞了。
