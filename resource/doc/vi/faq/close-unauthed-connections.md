# Tắt kết nối chưa được xác thực

**Câu hỏi:**

Làm thế nào để đóng kết nối của khách hàng không gửi dữ liệu trong khoảng thời gian nhất định, ví dụ 30 giây không nhận được dữ liệu nào thì tự động đóng kết nối của khách hàng này, mục đích là để các kết nối chưa được xác thực phải xác thực trong khoảng thời gian quy định.

**Trả lời:**

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('xxx://x.x.x.x:x');
$worker->onConnect = function(TcpConnection $connection)
{
    // Tạm thời thêm một thuộc tính auth_timer_id vào đối tượng $connection để lưu id của bộ định thời
    // Đóng kết nối sau 30 giây, cần client gửi xác thực để xóa bộ định thời
    $connection->auth_timer_id = Timer::add(30, function()use($connection){
        $connection->close();
    }, null, false);
};
$worker->onMessage = function(TcpConnection $connection, $msg)
{
    $msg = json_decode($msg, true);
    switch($msg['type'])
    {
    case 'login':
        ...Bỏ qua
        // Xác thực thành công, xóa bộ định thời để ngăn kết nối bị đóng
        Timer::del($connection->auth_timer_id);
        break;
         ... Bỏ qua
    }
    ... Bỏ qua
}
```
