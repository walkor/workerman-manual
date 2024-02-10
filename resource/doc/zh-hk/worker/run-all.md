# runAll
```php
void Worker::runAll(void)
```
運行所有Worker實例。

**注意：**

Worker::runAll()執行後將永久阻塞，也就是說位於Worker::runAll()後面的代碼將不會被執行。所有Worker實例化應該都在Worker::runAll()前進行。

### 參數
無參數

### 返回值
無返回

## 範例 運行多個Worker實例

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

// 運行所有Worker實例
Worker::runAll();
```


**注意：**

Windows版本的workerman不支持在同一個文件中實例化多個Worker。
上面的例子無法在Windows版本的workerman下運行。

Windows版本的workerman需要將多個Worker實例初始化放在不同的文件中，像下面這樣

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

// 運行所有Worker實例(這裡只有一個實例)
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

// 運行所有Worker實例(這裡只有一個實例)
Worker::runAll();
```
