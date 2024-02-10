# pipe
## 說明:
```php
void Connection::pipe(TcpConnection $target_connection)
```

## 參數
將當前連線的數據流導入目標連接。內置了流量控制。此方法做TCP代理非常有用

## 範例 TCP代理
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8483');
$worker->count = 12;

// tcp連接建立後
$worker->onConnect = function(TcpConnection $connection)
{
    // 建立本地80端口的異步連接
    $connection_to_80 = new AsyncTcpConnection('tcp://127.0.0.1:80');
    // 設置將當前客戶端連接的數據導向80端口的連接
    $connection->pipe($connection_to_80);
    // 設置80端口連接返回的數據導向客戶端連接
    $connection_to_80->pipe($connection);
    // 執行異步連接
    $connection_to_80->connect();
};

// 運行worker
Worker::runAll();
```
