# Các sự kiện đang được hỗ trợ bởi Workerman

| Tên  | Sự mở rộng phụ thuộc | Hỗ trợ coroutine | Ưu tiên | Phiên bản Workerman |
|-----|------|--|-----|
|  Workerman\Events\Select   |   Không   | Không hỗ trợ  |  Mặc định của nhân cốt   |  >=3.0  ｜
|  Workerman\Events\Revolt   |   event(được lựa chọn)   | Hỗ trợ |  Cần cài đặt [revolt/event-loop](https://github.com/revoltphp/event-loop)   |  >=5.0  |
|  Workerman\Events\Event   |   event   | Không hỗ trợ |  Mặc định của nhân cốt   |  >=3.0  |
|  Workerman\Events\Swoole   |  [swoole](https://github.com/swoole/swoole-src)   | Hỗ trợ |  Cần thiết lập thủ công   |  >=4.0  |
|  Workerman\Events\Swow   |   [swow](https://github.com/swow/swow)   | Hỗ trợ |  Cần thiết lập thủ công   |  >=5.0  |

* Mỗi loại nhân cốt sẽ cung cấp các tính năng độc lập, ví dụ sử dụng `Revolt` sẽ cho phép workerman hỗ trợ [Fiber coroutine (sợi)](https://www.php.net/manual/zh/language.fibers.php) được tích hợp sẵn trong PHP, việc sử dụng `Swoole` sẽ cho phép workerman hỗ trợ coroutine của Swoole
* Các sự kiện nhân cốt giao tranh lẫn nhau, ví dụ khi sử dụng coroutine Fiber của `Revolt`, bạn không thể sử dụng coroutine của Swoole hoặc Swow
* `Revolt` cần cài đặt `composer require revolt/event-loop ^1.0.0`, sau khi cài đặt, nhân cốt của workerman sẽ tự động thiết lập nó là nhân cốt sự kiện ưu tiên
* `Swoole` và `Swow` cần thiết lập thủ công `Worker::$eventLoopClass` để có hiệu lực (xem đoạn văn bản tiếp theo)
* Swoole mặc định không bật [Chế độ coroutine tự động](https://wiki.swoole.com/#/runtime?id=runtime), điều này có nghĩa là cuộc gọi I / O dựa trên Pdo, Redis, việc đọc / viết tập tin được tích hợp sẵn trong PHP vẫn sẽ là cuộc gọi chặn
* Để bật chế độ coroutine tự động cho swoole, cần phải gọi `\Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_ALL);` thủ công

> **Chú ý**
> Phần mở rộng swow sẽ tự động thay đổi hành vi của một số hàm tích hợp sẵn trong PHP, điều này có thể dẫn đến việc workerman không thể đáp ứng yêu cầu hoặc tín hiệu khi mở rộng swow được bật mà không sử dụng swow làm mô tả sự kiện, do đó nếu bạn không sử dụng swow như mô tả sự kiện ở mức độ thấp nhất, bạn cần phải comment swow trong tập tin php.ini

Xem thêm tại [workerman coroutine](../fiber.md)

# Cài đặt sự kiện đầu vào cho workerman

Dưới đây là cách cài đặt sự kiện đầu vào cho workerman một cách thủ công

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Swoole\Coroutine\Http\Client;
use Swoole\Coroutine;

// Cài đặt thủ công cho sự kiện đầu vào
Worker::$eventLoopClass = Workerman\Events\Revolt::class;
//Worker::$eventLoopClass = Workerman\Events\Select::class;
//Worker::$eventLoopClass = Workerman\Events\Event::class;
//Worker::$eventLoopClass = Workerman\Events\Swoole::class;
//Worker::$eventLoopClass = Workerman\Events\Swow::class;
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
