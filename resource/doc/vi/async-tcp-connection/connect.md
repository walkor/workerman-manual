# Phương thức connect

```php
void AsyncTcpConnection::connect()
```

Thực hiện kết nối bất đồng bộ. Phương thức này sẽ trả về ngay lập tức.

Lưu ý: Nếu cần thiết lập onError callback cho kết nối bất đồng bộ, thì cần thiết lập trước khi thực hiện connect, nếu không onError callback có thể không được kích hoạt. Ví dụ dưới đây onError callback có thể không được kích hoạt, không thể bắt sự kiện lỗi kết nối bất đồng bộ.

```php
$connection = new AsyncTcpConnection('tcp://baidu.com:81');
// Chưa thiết lập onError callback khi thực hiện kết nối
$connection->connect();
$connection->onError = function($connection, $err_code, $err_msg)
{
    echo "$err_code, $err_msg";
};
```

### Tham số
Không có tham số

### Giá trị trả về
Không có giá trị trả về

### Ví dụ về Proxy MySQL

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Địa chỉ MySQL thực, giả sử ở đây là cổng 3306 trên máy cục bộ
$REAL_MYSQL_ADDRESS = 'tcp://127.0.0.1:3306';

// Proxy lắng nghe cổng 4406 trên máy localhost
$proxy = new Worker('tcp://0.0.0.0:4406');

$proxy->onConnect = function(TcpConnection $connection)
{
    global $REAL_MYSQL_ADDRESS;
    // Thiết lập kết nối bất đồng bộ đến máy chủ MySQL thực tế
    $connection_to_mysql = new AsyncTcpConnection($REAL_MYSQL_ADDRESS);
    // Khi kết nối MySQL gửi dữ liệu, chuyển tiếp đến kết nối khách hàng tương ứng
    $connection_to_mysql->onMessage = function(AsyncTcpConnection $connection_to_mysql, $buffer)use($connection)
    {
        $connection->send($buffer);
    };
    // Khi kết nối MySQL đóng, đóng kết nối proxy tương ứng đến khách hàng
    $connection_to_mysql->onClose = function(AsyncTcpConnection $connection_to_mysql)use($connection)
    {
        $connection->close();
    };
    // Khi kết nối MySQL gặp lỗi, đóng kết nối proxy tương ứng đến khách hàng
    $connection_to_mysql->onError = function(AsyncTcpConnection $connection_to_mysql)use($connection)
    {
        $connection->close();
    };
    // Thực hiện kết nối bất đồng bộ
    $connection_to_mysql->connect();

    // Khi khách hàng gửi dữ liệu, chuyển tiếp đến kết nối MySQL tương ứng
    $connection->onMessage = function(TcpConnection $connection, $buffer)use($connection_to_mysql)
    {
        $connection_to_mysql->send($buffer);
    };
    // Khi kết nối khách hàng đóng, đóng kết nối MySQL tương ứng
    $connection->onClose = function(TcpConnection $connection)use($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };
    // Khi kết nối khách hàng gặp lỗi, đóng kết nối MySQL tương ứng
    $connection->onError = function(TcpConnection $connection)use($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };

};
// Chạy worker
Worker::runAll();
```

 **Kiểm tra**

```bash
mysql -uroot -P4406 -h127.0.0.1 -p
```
