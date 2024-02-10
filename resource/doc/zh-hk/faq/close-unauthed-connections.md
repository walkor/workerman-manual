# 關閉未認證連線
**問題：**

如何關閉在規定時間內未發送過數據的客戶端，
比如30秒內沒收到一條數據就自動關閉這個客戶端連線，
目的是為了讓未認證的連線必須在規定時間內認證

**答案：**

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('xxx://x.x.x.x:x');
$worker->onConnect = function(TcpConnection $connection)
{
    // 臨時給$connection對象添加一個auth_timer_id屬性存儲定時器id
    // 定時30秒關閉連線，需要客戶端30秒內發送驗證刪除定時器
    $connection->auth_timer_id = Timer::add(30, function()use($connection){
        $connection->close();
    }, null, false);
};
$worker->onMessage = function(TcpConnection $connection, $msg)
{
    $msg = json_decode($msg, true);
    switch($msg['type'])
    {
    case 'login':
        ...略
        // 驗證成功，刪除定時器，防止連接被關閉
        Timer::del($connection->auth_timer_id);
        break;
         ... 略
    }
    ... 略
}
```
