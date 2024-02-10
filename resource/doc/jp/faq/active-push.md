# メッセージをアクティブにプッシュする方法

1. タイマーを使用してデータを定期的にプッシュできます。
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:1234');
// プロセスが起動した後、クライアントに定期的にデータをプッシュ
$worker->onWorkerStart = function($worker){
    Timer::add(1, function()use($worker){
        foreach($worker->connections as $connection) {
            $connection->send('hello');
        }
    });
};
Worker::runAll();
```

2. 他のプロジェクトからWorkermanにデータをプッシュする必要がある場合
参照：[よくある質問-他のプロジェクトでのプッシュ](push-in-other-project.md)

