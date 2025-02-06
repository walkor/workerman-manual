# Locker 协程锁

Locker是一种内存锁，用于协程间的同步，常用来在协程中排队访问某种临界资源，例如某个数据库组件没有做连接池，则可以通过Locker来排队使用该组件，避免因为多个协程同时使用同一个连接资源导致数据异常。

> **提示**
> 此特性需要 workerman>=5.1.0

## 注意
* Locker支持Swoole/Swow/Fiber/Select/Event驱动
* Locker是用于同一个进程的不同协程间排队互斥访问某个资源的，进程与进程间互不影响


```php
<?php
use Workerman\Connection\TcpConnection;
use Workerman\Coroutine\Locker;
use Workerman\Events\Swoole;
use Workerman\Protocols\Http\Request;
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8001');

$worker->eventLoop = Swoole::class; // Or Swow::class or Fiber::class

$worker->onMessage = function (TcpConnection $connection, Request $request) {
    static $redis;
    if (!$redis) {
        $redis = new Redis();
        $redis->connect('127.0.0.1', 6379);
    }
    // 避免多个协程同时使用同一个连接，发生类似 "Socket#10 has already been bound to another coroutine" 错误
    Locker::lock('redis');
    $time = $redis->time();
    Locker::unlock('redis');
    $connection->send(json_encode($time));
};

Worker::runAll();

```

## 接口说明

```php
interface LockerInterface
{
    /**
     * 加锁
     */
    public static function lock(string $key): bool
    
    /**
     * 解锁
     */
    public static function unlock(string $key): bool
}
```