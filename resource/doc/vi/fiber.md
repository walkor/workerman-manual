# Cơ Sở Hạ Tầng Fiber
Từ phiên bản 5.0.0 trở đi, workerman hỗ trợ [Fiber (cơ sở hạ tầng)](https://www.php.net/manual/zh/language.fibers.php)

> **Chú ý**
> Để sử dụng tính năng Fiber, PHP cần phiên bản >= 8.1 và cài đặt `composer require revolt/event-loop ^1.0.0`

### Giới thiệu

Fiber là cơ sở hạ tầng coroutine (đoạn mã con), cho phép tạm dừng mã PHP, sau đó tiếp tục chạy khi cần thiết. Điều quan trọng nhất của nó là cho phép nhà phát triển viết mã bất đồng bộ không chặn một cách đồng bộ, làm tăng tính bảo trì của mã một cách đáng kể.

### Ví dụ
Dưới đây là một ví dụ so sánh cách viết mã coroutine và lập trình gọi lại bất đồng bộ.

**Lập trình gọi lại bất đồng bộ**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Http\Client;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    static $http;
    $http = $http ?: new Client();
    // Gọi tới giao diện HTTP
    $http->get('http://example.com/', function ($response) use ($connection) {
        // Trì hoãn gửi trong 1 giây
        Timer::add(1, function() use ($connection, $response) {
            // Gửi dữ liệu tới trình duyệt
            $connection->send((string)$response->getBody());
        }, null, false);
    });
};

Worker::runAll();
```

**Sử dụng coroutine**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Http\Client;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    static $http;
    $http = $http ?: new Client();
    // Gọi tới giao diện HTTP
    $response = $http->get('http://example.com/');
    // Trì hoãn 1 giây
    Timer::sleep(1);
    // Gửi dữ liệu
    $connection->send((string)$response->getBody());
};

Worker::runAll();
```

> **Chú ý**
> Đoạn mã trên cần cài đặt `composer require workerman/http-client ^2.0.0`

Cả hai cách viết trên đều là chạy bất đồng bộ, hiệu suất chạy đều tốt, nhưng cách sử dụng coroutine dễ đọc và bảo trì hơn so với lập trình gọi lại bất đồng bộ.

### Lưu ý về Fiber
* Coroutine Fiber không hỗ trợ coroutine hóa các hàm Pdo, Redis và hàm chặn nội bộ của PHP, tức là sử dụng các phần mở rộng và hàm này vẫn sẽ là cuộc gọi chặn
* Hiện tại, các client coroutine Fiber có sẵn bao gồm [workerman/http-client](../components/workerman-http-client.md), [workerman/redis](../components/workerman-redis.md)

# Coroutine Swoole
workerman v5 cũng hỗ trợ sử dụng Swoole như là cơ sở hạ tầng sự kiện dẫn đầu

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Swoole\Coroutine\Http\Client;
use Swoole\Coroutine;

// Ở đây, cần thiết đặt Swoole thành cơ sở hạ tầng sự kiện dẫn đầu
Worker::$eventLoopClass = Workerman\Events\Swoole::class;
$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    Coroutine::create(function() use ($connection) {
        $cli = new Client('example.com', 80);
        $cli->get('/get');
        $connection->send($cli->body);
    });
};

Worker::runAll();
```
**Lưu ý**
* Nên sử dụng phiên bản swoole 5.0 hoặc cao hơn
* Sử dụng Swoole như cơ sở hạ tầng sự kiện dẫn đầu cho phép workerman hỗ trợ coroutine của swoole
* Khi sử dụng Swoole như cơ sở hạ tầng sự kiện dẫn đầu, không cần cài đặt extension event
* Mặc định, swoole không mở coroutine chỉ bấm một phím, tức là sử dụng Pdo, Redis, đọc/ghi tập tin nội bộ của PHP là cuộc gọi chặn
* Nếu muốn bật coroutine bằng một phím, cần gọi `\Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_ALL);`

Xem thêm tại [Swoole thủ thuật](https://wiki.swoole.com/)

Xem thêm tại [Sự kiện dẫn đầu](appendices/event.md)

# Về Coroutine
Trước hết, không cần quá mê tín vào việc sử dụng coroutine. Khi cơ sở dữ liệu, Redis và các lưu trữ trong mạng nội bộ, việc sử dụng nhiều tiến trình chặn thường nhanh hơn coroutine. Từ dữ liệu thử nghiệm của [techempower.com trong 3 năm](https://www.techempower.com/benchmarks/#section=data-r21&l=zik073-6bj&test=db), ta có thể thấy hiệu suất gọi chặn cơ sở dữ liệu của workerman vượt trội so với coroutine và pool kết nối cơ sở dữ liệu của swoole, thậm chí còn cao hơn gần 1 lần so với khung coroutine như gin, echo trong ngôn ngữ go.

Workerman đã nâng cấp hiệu suất ứng dụng PHP lên nhiều lần, đa số các dự án workerman thêm vào coroutine có thể không tăng hiệu suất nhiều hơn.
Nếu hệ thống của bạn có cuộc gọi chậm, ví dụ có cuộc gọi HTTP bên ngoài, bạn có thể xem xét sử dụng coroutine để nâng cao hiệu suất.




