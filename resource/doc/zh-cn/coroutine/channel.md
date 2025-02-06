# Channel 协程通道

Channel是协程之间通信的一种机制。一个协程可以将数据推送到通道中，而另一个协程可以从中弹出数据，从而实现协程之间的同步和数据共享。

> **提示**
> 此特性需要 workerman>=5.1.0

## 注意
* 底层自动支持Swoole/Swow/Fiber/Select/Event驱动
* 当使用Select/Event驱动时，不支持pop/push的超时参数

```php
<?php
use Workerman\Connection\TcpConnection;
use Workerman\Coroutine\Channel;
use Workerman\Coroutine\Coroutine;
use Workerman\Events\Swoole;
use Workerman\Protocols\Http\Request;
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Http Server
$worker = new Worker('http://0.0.0.0:8001');
$worker->eventLoop = Swoole::class; // Or Swow::class or Fiber::class
$worker->onMessage = function (TcpConnection $connection, Request $request) {
    $channel = new Channel(2);
    Coroutine::create(function () use ($channel) {
        $channel->push('Task 1 Done');
    });
    Coroutine::create(function () use ($channel) {
        $channel->push('Task 2 Done');
    });
    $result = [];
    for ($i = 0; $i < 2; $i++) {
        $result[] = $channel->pop();
    }
    $connection->send(json_encode($result)); // Response: ["Task 1 Done","Task 2 Done"]
};
Worker::runAll();
```

## 接口说明

```php
interface ChannelInterface
{
    /**
     * 将数据推送到通道中，支持超时(单位秒)，超时返回false
     */
    public function push(mixed $data, float $timeout = -1): bool;

    /**
     * 从通道中弹出数据，支持超时(超时单位秒)，超时返回false
     */
    public function pop(float $timeout = -1): mixed;

    /**
     * 获取通道中数据的长度
     */
    public function length(): int;

    /**
     * 获取通道的容量
     */
    public function getCapacity(): int;

    /**
     * 是否有消费者，即是否有协程在等待pop数据
     */
    public function hasConsumers(): bool;

    /**
     * 是否有生产者，即是否有协程在等待push数据到通道
     */
    public function hasProducers(): bool;

    /**
     * 关闭通道
     */
    public function close(): void;

}
```

