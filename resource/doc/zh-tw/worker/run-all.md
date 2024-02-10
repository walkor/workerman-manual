# runAll
```php
void Worker::runAll(void)
```
執行所有 Worker 實例。

**注意：**

Worker::runAll() 執行後將永久阻塞，也就是說位於 Worker::runAll() 後面的程式碼將不會被執行。所有 Worker 實例化應該都在 Worker::runAll() 前進行。

### 參數
無參數

### 返回值
無返回

## 範例 執行多個 Worker 實例

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

// 執行所有 Worker 實例
Worker::runAll();
```

**注意：**

Windows 版本的 Workerman 不支援在同一個檔案中實例化多個 Worker。上面的例子無法在 Windows 版本的 Workerman 下執行。

Windows 版本的 Workerman 需要將多個 Worker 實例初始化放在不同的檔案中，像下面這樣：

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

// 執行所有 Worker 實例(這裡只有一個實例)
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

// 執行所有 Worker 實例(這裡只有一個實例)
Worker::runAll();
```

