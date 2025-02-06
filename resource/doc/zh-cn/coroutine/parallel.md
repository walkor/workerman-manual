# Parallel

Parallel 是一个并行任务调度工具，允许在程序中同时执行多个异步任务，并在所有任务完成后获取结果。Parallel是基于Barrier实现的。


> **注意**
> 底层自动识别驱动类型，仅支持Swoole/Swow/Fiber驱动

> **提示**
> 此特性需要 workerman>=5.1.0


```php
<?php
use Workerman\Connection\TcpConnection;
use Workerman\Coroutine\Parallel;
use Workerman\Events\Swoole;
use Workerman\Protocols\Http\Request;
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Http Server
$worker = new Worker('http://0.0.0.0:8001');
$worker->eventLoop = Swoole::class; // Or Swow::class or Fiber::class
$worker->onMessage = function (TcpConnection $connection, Request $request) {
    $parallel = new Parallel();
    for ($i=1; $i<5; $i++) {
        $parallel->add(function () use ($i) {
            // Do something
            return $i;
        });
    }
    $results = $parallel->wait();
    $connection->send(json_encode($results)); // Response: [1,2,3,4]
};
Worker::runAll();
```

## 接口说明

```php
interface ParallelInterface
{
    /**
     * 构造函数，$concurrent为并行任务数，-1表示不限制并行任务数
     */
    public function __construct(int $concurrent = -1);

    /**
     * 添加一个并行任务
     */
    public function add(callable $callable, ?string $key = null): void;

    /**
     * 等待所有任务完成并返回结果
     */
    public function wait(): array;

    /**
     * 获取任务中发生异常的结果
     */
    public function getExceptions(): array;
}
```

