# 未認証の接続を閉じる
**質問:**

30秒間データを送信しなかったクライアントを自動的に閉じる方法について、
例えば、30秒以内に認証されていない接続は規定の時間内に認証する必要があります。

**回答:**

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('xxx://x.x.x.x:x');
$worker->onConnect = function(TcpConnection $connection)
{
    // 一時的に$connectionオブジェクトにauth_timer_idプロパティを追加して、タイマーIDを格納します
    // 30秒後に接続を閉じるタイマーを追加します。クライアントは30秒以内に認証しないとタイマーが削除されます
    $connection->auth_timer_id = Timer::add(30, function() use ($connection){
        $connection->close();
    }, null, false);
};
$worker->onMessage = function(TcpConnection $connection, $msg)
{
    $msg = json_decode($msg, true);
    switch($msg['type'])
    {
    case 'login':
        ...省略
        // 認証に成功したらタイマーを削除し、接続が閉じられるのを防ぎます
        Timer::del($connection->auth_timer_id);
        break;
         ... 省略
    }
    ... 省略
}
```
