# Mở cổng 843 cho Flash

Khi Flash khởi tạo kết nối socket với máy chủ từ xa, trước hết nó sẽ yêu cầu một tệp chính sách an toàn từ cổng 843 của máy chủ tương ứng. Nếu không, Flash sẽ không thể thiết lập kết nối với máy chủ. Trong Workerman, bạn có thể mở một cổng 843 và trả về tệp chính sách an toàn như sau.

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$flash_policy = new Worker('tcp://0.0.0.0:843');
$flash_policy->onMessage = function(TcpConnection $connection, $message)
{
    $connection->send('<?xml version="1.0"?><cross-domain-policy><site-control permitted-cross-domain-policies="all"/><allow-access-from domain="*" to-ports="*"/></cross-domain-policy>'."\0");
};

if(!defined('GLOBAL_START'))
{
    Worker::runAll();
}
```

Nội dung chính sách an toàn XML có thể được tùy chỉnh theo nhu cầu của bạn.
