# runAll

```php
void Worker::runAll(void)
```
Chạy tất cả các instance của Worker.

**Lưu ý:**

Sau khi thực thi Worker::runAll(), chương trình sẽ bị chặn vĩnh viễn, có nghĩa là đoạn code sau Worker::runAll() sẽ không được thực thi. Tất cả các instance của Worker phải được khởi tạo trước Worker::runAll().

### Tham số
Không có tham số

### Giá trị trả về
Không có giá trị trả về

## Ví dụ: Chạy nhiều instance của Worker

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

// Chạy tất cả các instance của Worker
Worker::runAll();
```

**Lưu ý:**

Phiên bản Windows của workerman không hỗ trợ việc khởi tạo nhiều instance của Worker trong cùng một tệp.
Ví dụ trên sẽ không hoạt động trong phiên bản Windows của workerman.

Phiên bản Windows của workerman yêu cầu khởi tạo nhiều instance của Worker được đặt trong các tệp khác nhau, giống như sau đây

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

// Chạy tất cả các instance của Worker (chỉ có một instance ở đây)
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

// Chạy tất cả các instance của Worker (chỉ có một instance ở đây)
Worker::runAll();
```
