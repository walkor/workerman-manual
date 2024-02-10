# パイプ
## 説明：
```php
void Connection::pipe(TcpConnection $target_connection)
```

## パラメータ
現在の接続のデータストリームをターゲット接続にリダイレクトします。フロー制御が組み込まれています。このメソッドはTCPプロキシに非常に便利です。

## TCPプロキシの例

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8483');
$worker->count = 12;

// TCP接続後
$worker->onConnect = function(TcpConnection $connection)
{
    // ローカルの80ポートに非同期接続を確立する
    $connection_to_80 = new AsyncTcpConnection('tcp://127.0.0.1:80');
    // 現在のクライアント接続のデータを80ポートの接続にリダイレクトするように設定する
    $connection->pipe($connection_to_80);
    // 80ポート接続から返されたデータをクライアント接続にリダイレクトするように設定する
    $connection_to_80->pipe($connection);
    // 非同期接続を実行する
    $connection_to_80->connect();
};

// ワーカーを実行する
Worker::runAll();
```
