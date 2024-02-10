# sleep
```php
int \Workerman\Timer::sleep(float $delay)
```
Tương tự như hàm `sleep()` tích hợp sẵn trong PHP, khác biệt là `Timer::sleep()` là không chặn (non-blocking) (không chặn tiến trình hiện tại)

> **Chú ý**
> Tính năng này yêu cầu workerman>=5.0
> Tính năng này yêu cầu cài đặt composer require revolt/event-loop ^1.0.0, hoặc sử dụng Swoole/Swow làm động cơ sự kiện


### Tham số
 ``` delay ```

Thời gian giữa mỗi lần thực thi, tính bằng giây, hỗ trợ số thập phân, có thể chính xác đến 0.001, tức là có thể chính xác đến mili giây.

### Giá trị trả về
Không có giá trị trả về

### Ví dụ

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    // Trì hoãn gửi dữ liệu sau 1.5 giây
    Timer::sleep(1.5);
    // Gửi dữ liệu
    $connection->send('Xin chào workerman');
};

Worker::runAll();
```
