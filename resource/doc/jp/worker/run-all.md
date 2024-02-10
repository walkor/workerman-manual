# runAll
```php
void Worker::runAll(void)
```
すべてのWorkerインスタンスを実行します。

**注意：**

Worker::runAll()を実行すると、永久的にブロックされます。つまり、Worker::runAll()の後にあるコードは実行されません。すべてのWorkerインスタンスの初期化はWorker::runAll()の前に行う必要があります。

### パラメーター
パラメーターなし

### 戻り値
戻り値なし

## 例　複数のWorkerインスタンスを実行する

start.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello http');
};

$ws_worker = new Worker('websocket://0.0.0.0:4567');
$ws_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello websocket');
};

// すべてのWorkerインスタンスを実行
Worker::runAll();
```


**注意：**

Windows版のworkermanは同じファイル内で複数のWorkerをインスタンス化することをサポートしていません。
上記の例はWindows版のworkermanでは実行できません。

Windows版のworkermanでは、複数のWorkerインスタンスの初期化を以下のように別々のファイルに配置する必要があります。

start_http.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello http');
};

// すべてのWorkerインスタンスを実行（ここには1つのインスタンスのみがあります）
Worker::runAll();
```

start_websocket.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$ws_worker = new Worker('websocket://0.0.0.0:4567');
$ws_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello websocket');
};

// すべてのWorkerインスタンスを実行（ここには1つのインスタンスのみがあります）
Worker::runAll();
```
