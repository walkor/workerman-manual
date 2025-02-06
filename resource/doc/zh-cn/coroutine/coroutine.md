## 协程

协程是一种比线程更轻量级的用户级并发机制，能够在进程中实现多任务调度。它通过手动控制挂起和恢复来实现协程间的切换，避免了进程上下文切换的开销。
workerman提供了一个通用的协程接口，底层自动兼容Swoole/Swow/Fiber驱动。

> **提示**
> 此特性需要 workerman>=5.1.0

## 注意
* 仅支持Swoole/Swow/Fiber驱动
* 通过Swoole或者Swow扩展可以实现PHP阻塞函数自动协程化，从而实现原来的同步代码异步执行
* Fiber无法像Swoole和Swow那样自动协程化，遇到PHP自带的阻塞函数时会阻塞整个进程，并不会发生协程切换
* 如果使用Fiber协程需要安装 `composer require revolt/event-loop`
* 当使用Swoole/Swow/Fiber驱动时，workerman每次运行onWorkerStart/onMessage/onConnect/onClose等回调时会自动创建一个协程来执行
* 可以利用`$worker->eventLoop=xxx;`给每个worker设置不同的协程驱动

```php
<?php
use Workerman\Connection\TcpConnection;
use Workerman\Coroutine\Coroutine;
use Workerman\Events\Swoole;
use Workerman\Protocols\Http\Request;
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker1 = new Worker('http://0.0.0.0:8001');
$worker1->eventLoop = Swoole::class; // 使用Swoole协程
$worker1->onMessage = function (TcpConnection $connection, Request $request) {
    Coroutine::create(function () {
        echo file_get_contents("http://www.example.com/event/notify");
    });
    $connection->send('ok');
};

$worker2 = new Worker('http://0.0.0.0:8001');
$worker2->eventLoop = Fiber::class; // 使用自带的Fiber协程
$worker2->onMessage = function (TcpConnection $connection, Request $request) {
    Coroutine::create(function () {
        echo file_get_contents("http://www.example.com/event/notify");
    });
    $connection->send('ok');
};

Worker::runAll();
```

## 协程提供的接口

```php
interface CoroutineInterface
{

    /**
     * 创建协程并立即执行
     */
    public static function create(callable $callable, ...$data): CoroutineInterface;

    /**
     * 开始协程运行
     */
    public function start(mixed ...$args): mixed;

    /**
     * 恢复协程运行
     */
    public function resume(mixed ...$args): mixed;

    /**
     * 获取协程id
     */
    public function id(): int;

    /**
     * 设置协程销毁时的回调
     */
    public static function defer(callable $callable): void;

    /**
     * 暂停当前协程
     */
    public static function suspend(mixed $value = null): mixed;

    /**
     * 获取当前协程
     */
    public static function getCurrent(): CoroutineInterface|Fiber|SwowCoroutine|static;

    /**
     * 判断当前是否是协程环境
     */
    public static function isCoroutine(): bool;

}
```

## 关于协程

#### 优势
PHP引入协程后最大的作用就是可以用同步的方式编写异步代码，避免了回调地狱，提高了代码的可读性和可维护性。
协程能大幅度提升IO密集型业务的弹性，可以用较少的进程提供更大的吞吐量。

#### 劣势
但是引入协程后开发者需要时刻注意全局变量污染、资源竞争、第三方库改造等问题，开发维护成本增大，心智负担明显增加。

引入协程后产生了协程创建、调度、销毁、连接池等额外开销。
通过大量压测数据来看，在充分利用CPU的情况下，引入协程后极限性能比阻塞式IO下降约10%-20%。






