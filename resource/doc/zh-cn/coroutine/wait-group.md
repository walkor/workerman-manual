# 协程等待组 WaitGroup

`WaitGroup`与`Barrier`相似，是一个用于协程同步的工具，允许在异步任务中等待所有协程执行完成后再继续后续逻辑。

与`Barrier`区别的是`WaitGroup`可以由开发者自行控制计数的增加和减少。

> **注意**
> 底层自动识别驱动类型，仅支持Swoole/Swow/Fiber驱动

> **提示**
> 此特性需要 workerman>=5.1.0

```php
<?php
use Workerman\Connection\TcpConnection;
use Workerman\Coroutine\WaitGroup;
use Workerman\Coroutine;
use Workerman\Events\Swoole;
use Workerman\Protocols\Http\Request;
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Http Server
$worker = new Worker('http://0.0.0.0:8001');
$worker->eventLoop = Swoole::class; // Or Swow::class or Fiber::class
$worker->onMessage = function (TcpConnection $connection, Request $request) {
    $wg = new WaitGroup();
    for ($i=1; $i<5; $i++) {
        $wg->add();
        Coroutine::create(function () use ($wg, $i) {
            try {
            // Do something
            } finally {
                $wg->done();
            }
        });
    }
    // Wait all coroutine done, timout after 10 seconds
    $result = $wg->wait(10.0);
    if (!$result) {
        $connection->send('WaitGroup Timeout');
        return;
    }
    $connection->send('All Task Done');
};

Worker::runAll();
```

## 接口说明

```php
interface WaitGroupInterface
{

    /**
     * 增加计数
     *
     * @param int $delta
     * @return bool
     */
    public function add(int $delta = 1): bool;

    /**
     * 完成计数
     *
     * @return bool
     */
    public function done(): bool;

    /**
     * 返回计数
     *
     * @return int
     */
    public function count(): int;

    /**
     * 协程等待
     *
     * @param int|float $timeout second
     * @return bool timeout:false success:true
     */
    public function wait(int|float $timeout = -1): bool;
}
```
