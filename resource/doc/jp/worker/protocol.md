# プロトコル
要求 ```（workerman >= 3.2.7）```

## 説明:
```php
string Worker::$protocol
```

現在のWorkerインスタンスのプロトコルクラスを設定します。

注：プロトコル処理クラスは、Workerの初期化時に直接指定することができます。例えば
```php
$worker = new Worker('http://0.0.0.0:8686');
```



## 例


```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8686');
$worker->protocol = 'Workerman\\Protocols\\Http';

$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    $connection->send("hello");
};

// workerを実行
Worker::runAll();
```

上記のコードは、以下のコードと同等です


```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

/**
 * まず、ユーザーが独自の\Protocols\Httpプロトコルクラスを持っているかどうかを確認し、
 * 持っていない場合はworkermanの内蔵プロトコルクラスWorkerman\Protocols\Httpを使用します。
 */
$worker = new Worker('http://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    $connection->send("hello");
};

// workerを実行
Worker::runAll();
```
