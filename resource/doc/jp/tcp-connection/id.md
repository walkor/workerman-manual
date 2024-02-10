# id

## 説明:
```php
int Connection::$id
```

接続のIDです。これは自動的に増加する整数です。

注意：workermanは複数のプロセスを持っており、各プロセス内で独自の接続IDを維持しています。そのため、複数のプロセス間で接続IDが重複することがあります。
重複しない接続IDが必要な場合は、必要に応じてconnection->idにworker->idのプレフィックスを追加することができます。

## 参照
[Workerのconnectionsプロパティ](../worker/connections.md)

## 例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo $connection->id;
};
// workerを実行
Worker::runAll();
```
