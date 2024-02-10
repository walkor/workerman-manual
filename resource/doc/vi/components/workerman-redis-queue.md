# workerman/redis-queue

Hàng đợi tin nhắn dựa trên Redis, hỗ trợ xử lý tin nhắn trì hoãn.

## Địa chỉ dự án:
https://github.com/walkor/redis-queue

## Cài đặt:
```
composer require workerman/redis-queue
```

## Ví dụ
```
<?php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\RedisQueue\Client;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function () {
    $client = new Client('redis://127.0.0.1:6379');
   // Đăng ký
    $client->subscribe('user-1', function($data){
        echo "user-1\n";
        var_export($data);
    });
   // Đăng ký
    $client->subscribe('user-2', function($data){
        echo "user-2\n";
        var_export($data);
    });
    // Gửi tin nhắn tới hàng đợi theo định kỳ
    Timer::add(1, function()use($client){
        $client->send('user-1', ['some', 'data']);
    });
};

Worker::runAll();
```

## API
  * <a href="#construct"><code>Client::<b>__construct()</b></code></a>
  * <a href="#send"><code>Client::<b>send()</b></code></a>
  * <a href="#subscribe"><code>Client::<b>subscribe()</b></code></a>
  * <a href="#unsubscribe"><code>Client::<b>unsubscribe()</b></code></a>

-------------------------------------------------------

<a name="construct"></a>
### __construct (string $address, [array $options])

Tạo một thể hiện

  * `$address`  Giống như `redis://ip:6379`, phải bắt đầu bằng redis.

  * `$options`  Bao gồm các tùy chọn sau:
    * `auth`: Thông tin xác thực, mặc định ''
    * `db`: db, mặc định 0
    * `max_attempts`: Số lần thử sau khi thất bại, mặc định 5
    * `retry_seconds`: Khoảng thời gian giữa các lần thử, tính bằng giây. Mặc định 5

> Thất bại trong quá trình tiêu thụ có nghĩa là doanh nghiệp ném ngoại lệ `Exception` hoặc `Error`. Sau khi thất bại, tin nhắn sẽ được đặt trong hàng đợi trì hoãn để chờ thử lại, số lần thử lại được điều khiển bởi `max_attempts`, khoảng thời gian thử lại được điều khiển bởi `retry_seconds` và `max_attempts`. Ví dụ, nếu `max_attempts` là 5, `retry_seconds` là 10, khoảng thời gian thử lại lần thứ 1 là `1*10` giây, khoảng thời gian thử lại lần thứ 2 là `2*10` giây, khoảng thời gian thử lại lần thứ 3 là `3*10` giây, và cứ tiếp tục như vậy đến khi thử lại 5 lần. Nếu vượt quá số lần thử do `max_attempts`, tin nhắn sẽ được đặt vào hàng đợi thất bại có khóa là `{redis-queue}-failed`(trước phiên bản 1.0.5 là `redis-queue-failed`)

-------------------------------------------------------

<a name="send"></a>
### send(String $queue, Mixed $data, [int $dely=0])

Gửi một tin nhắn tới hàng đợi

* `$queue` Tên hàng đợi, kiểu `String`
* `$data` Tin nhắn cụ thể được gửi, có thể là mảng hoặc chuỗi, kiểu `Mixed`
* `$dely` Thời gian trì hoãn tiêu thụ, tính bằng giây, mặc định 0, kiểu `Int`
  
-------------------------------------------------------

<a name="subscribe"></a>
### subscribe(mixed $queue, callable $callback)

Đăng ký một hoặc nhiều hàng đợi

* `$queue` Tên hàng đợi, có thể là chuỗi hoặc mảng chứa nhiều tên hàng đợi
* `$callback` Hàm gọi lại, định dạng là  `function (Mixed $data)` trong đó `$data` chính là `$data` trong hàm `send($queue, $data)`.

-------------------------------------------------------

<a name="unsubscribe"></a>
### unsubscribe(mixed $queue)

Hủy đăng ký

* `$queue` Tên hàng đợi hoặc một mảng chứa nhiều tên hàng đợi

-------------------------------------------------------

## Gửi tin nhắn tới hàng đợi trong môi trường không phải là workerman
Đôi khi, một số dự án chạy trên môi trường apache hoặc php-fpm không thể sử dụng dự án workerman/redis-queue, có thể tham khảo hàm sau để thực hiện gửi
```
function redis_queue_send($redis, $queue, $data, $delay = 0) {
    $queue_waiting = '{redis-queue}-waiting'; //trước phiên bản 1.0.5 là redis-queue-waiting
    $queue_delay = '{redis-queue}-delayed';//trước phiên bản 1.0.5 là redis-queue-delayed
    
    $now = time();
    $package_str = json_encode([
        'id'       => rand(),
        'time'     => $now,
        'delay'    => $delay,
        'attempts' => 0,
        'queue'    => $queue,
        'data'     => $data
    ]);
    if ($delay) {
        return $redis->zAdd($queue_delay, $now + $delay, $package_str);
    }
    return $redis->lPush($queue_waiting.$queue, $package_str);
}
```
Trong đó, tham số `$redis` là một thể hiện của redis. Ví dụ về cách sử dụng phần mở rộng redis như sau:
```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);
$queue = 'user-1';
$data= ['some', 'data'];
redis_queue_send($redis, $queue, $data);
````
