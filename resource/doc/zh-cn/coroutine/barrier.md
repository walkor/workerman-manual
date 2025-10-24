# 协程屏障 Barrier

Barrier 是一个用于协程同步的工具，允许在异步任务中等待所有协程执行完成后再继续后续逻辑。Barrier是基于PHP引用计数实现的。

> **注意**
> 底层自动识别驱动类型，仅支持Swoole/Swow/Fiber驱动

> **提示**
> 此特性需要 workerman>=5.1.0


```php
<?php
use Workerman\Connection\TcpConnection;
use Workerman\Coroutine\Barrier;
use Workerman\Coroutine;
use Workerman\Events\Swoole;
use Workerman\Protocols\Http\Request;
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Http Server
$worker = new Worker('http://0.0.0.0:8001');
$worker->eventLoop = Swoole::class; // Or Swow::class or Fiber::class
$worker->onMessage = function (TcpConnection $connection, Request $request) {
    $barrier = Barrier::create();
    for ($i=1; $i<5; $i++) {
        // 注意需要用use传递屏障
        Coroutine::create(function () use ($barrier, $i) {
            // Do something
        });
    }
    // Wait all coroutine done
    Barrier::wait($barrier);
    $connection->send('All Task Done');
};

Worker::runAll();
```


> **注意**
> 需要在协程用使用 use 语法传递屏障，增加引用计数。

## 接口说明

```php
interface BarrierInterface
{
    /**
     * 创建一个新的Barrier实例
     */
    public static function create(): object;
    
    /**
     * 暂停当前协程，等待指定Barrier中的所有协程任务完成
     */
    public static function wait(object &$barrier, int $timeout = -1): void;
}
```
