# 協定

## 說明:
```php
string Connection::$protocol
```

設置當前連接的協定類別


## 範例


```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    $connection->protocol = 'Workerman\\Protocols\\Http';
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    // send 時會自動調用$connection->protocol::encode()，打包數據後再發送
    $connection->send("hello");
};
// 運行worker
Worker::runAll();
```
