# stopAll
```php
void Worker::stopAll(void)
```

Dừng quá trình hiện tại và thoát.

> **Lưu ý**
> `Worker::stopAll()` được sử dụng để dừng quá trình hiện tại. Sau khi quá trình hiện tại thoát, quá trình chính sẽ ngay lập tức khởi chạy lại một quá trình mới. Nếu bạn muốn dừng dịch vụ workerman toàn bộ, hãy gọi `posix_kill(posix_getppid(), SIGINT)`.

### Tham số
Không có tham số



### Giá trị trả về
Không có giá trị trả về

## Ví dụ max_request

Dưới đây là ví dụ: sau khi xử lý 1000 yêu cầu, tiến trình con sẽ thực hiện stopAll để thoát, để khởi động lại một tiến trình hoàn toàn mới. Tương tự thuộc tính max_request của php-fpm, chủ yếu được sử dụng để giải quyết vấn đề rò rỉ bộ nhớ do lỗi mã nguồn kinh doanh php.

start.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Mỗi quá trình chỉ thực hiện tối đa 1000 yêu cầu
define('MAX_REQUEST', 1000);

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Số lượng yêu cầu đã xử lý
    static $request_count = 0;

    $connection->send('hello http');
    // Nếu số lượng yêu cầu đạt đến 1000
    if (++$request_count >= MAX_REQUEST)
    {
        /*
         * Thoát khỏi quá trình hiện tại, quá trình chính sẽ ngay lập tức khởi chạy một quá trình hoàn toàn mới để thay thế
         * Nhằm hoàn tất việc khởi động lại quá trình
         */
        Worker::stopAll();
    }
};

Worker::runAll();
```
