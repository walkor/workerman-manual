# プロトコル

## 説明:
```php
string Connection::$protocol
```

現在の接続のプロトコルクラスを設定します。

## 例

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
    // send時には自動的に$connection->protocol::encode()が呼び出され、データをパッケージ化して送信されます
    $connection->send("hello");
};
// workerを実行する
Worker::runAll();
```
