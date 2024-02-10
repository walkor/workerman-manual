# 協議

## 說明：
```php
string Connection::$protocol
```

設置當前連接的協議類別


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
    // 在發送時將自動調用 $connection->protocol::encode()，將數據打包後再發送
    $connection->send("hello");
};
// 運行 worker
Worker::runAll();
```
