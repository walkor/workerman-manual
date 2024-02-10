# 關閉未認證連線
**問題：**

如何在一定時間內未接收到數據的客戶端自動關閉連接，例如在30秒內未收到任何數據就自動關閉該客戶端的連接，目的是為了讓未認證的連接必須在指定時間內進行認證。

**答案：**

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('xxx://x.x.x.x:x');
$worker->onConnect = function(TcpConnection $connection)
{
    // 暫時給$connection對象添加一個auth_timer_id屬性來存儲定時器id
    // 設置30秒的定時器來關閉連接，需要客戶端在30秒內發送驗證數據以刪除定時器
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
        // 驗證成功，刪除定時器，以防止連接被關閉
        Timer::del($connection->auth_timer_id);
        break;
         ... 略
    }
    ... 略
}
```
