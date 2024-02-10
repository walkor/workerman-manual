# Debug cơ bản

WorkerMan có hai chế độ chạy, chế độ debug và chế độ chạy daemon.

Chạy ```php start.php start``` để vào chế độ debug, khi đó các hàm như ```echo, var_dump, var_export``` trong mã nguồn sẽ hiển thị trên terminal. Lưu ý rằng nếu chạy WorkerMan bằng ```php start.php start```, tất cả các tiến trình sẽ thoát khi terminal đóng.

Còn nếu chạy ```php start.php start -d``` thì sẽ chuyển sang chế độ daemon, tức là chế độ chạy chính thức trên môi trường sản xuất, và không bị ảnh hưởng khi đóng terminal.

Nếu muốn khi chạy ở chế độ daemon cũng có thể thấy được các hàm như ```echo, var_dump, var_export```, bạn có thể thiết lập thuộc tính Worker::$stdoutFile, ví dụ:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// In ra màn hình đầu ra vào tệp được chỉ định bởi Worker::$stdoutFile
Worker::$stdoutFile = '/tmp/stdout.log';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello world');
};

Worker::runAll();
```

Như vậy, tất cả các hàm như ```echo, var_dump, var_export``` sẽ được ghi vào tệp được chỉ định bởi ```Worker::$stdoutFile```. Lưu ý rằng đường dẫn được chỉ định bởi ```Worker::$stdoutFile``` phải có quyền ghi.
