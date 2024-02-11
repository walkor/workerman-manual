# Object và Persistence của Tài Nguyên

Trong phát triển Web truyền thống, các đối tượng, dữ liệu và tài nguyên được tạo ra bởi PHP sẽ được giải phóng hoàn toàn sau khi yêu cầu kết thúc, điều này gây khó khăn trong việc duy trì. Tuy nhiên, với Workerman, điều này có thể dễ dàng thực hiện.

Trong Workerman, nếu bạn muốn lưu trữ một số dữ liệu tài nguyên trong bộ nhớ vĩnh viễn, bạn có thể đặt tài nguyên vào biến toàn cục hoặc thành viên tĩnh của lớp.

Ví dụ đoạn code sau:

Sử dụng biến toàn cục ```$connection_count``` để lưu trữ số lượng kết nối của khách hàng trong quá trình hiện tại.

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Biến toàn cục, lưu trữ số lượng kết nối của khách hàng trong quá trình hiện tại
$connection_count = 0;

$worker = new Worker('tcp://0.0.0.0:1236');

$worker->onConnect = function(TcpConnection $connection)
{
    // Khi có kết nối mới từ khách hàng, số lượng kết nối +1
    global $connection_count;
    ++$connection_count;
    echo "now connection_count=$connection_count\n";
};

$worker->onClose = function(TcpConnection $connection)
{
    // Khi khách hàng đóng kết nối, số lượng kết nối -1
    global $connection_count;
    $connection_count--;
    echo "now connection_count=$connection_count\n";
};
```


## Phạm vi biến trong PHP xem tại:
https://php.net/manual/zh/language.variables.scope.php
