# Context 协程上下文

Context用于在协程中存储和传递上下文信息，例如数据库连接、用户信息等。每个协程有自己的上下文，不同协程之间的上下文是隔离的。

> **注意**
> 底层自动识别驱动类型，仅支持Swoole/Swow/Fiber驱动

> **提示**
> 此特性需要 workerman>=5.1.0


```php
<?php
<?php

use Workerman\Connection\TcpConnection;
use Workerman\Coroutine;
use Workerman\Coroutine\Context;
use Workerman\Events\Swoole;
use Workerman\Protocols\Http\Request;
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8001');

$worker->eventLoop = Swoole::class; // Or Swow::class or Fiber::class

$worker->onMessage = function (TcpConnection $connection, Request $request) {
    // 在当前协程设置context数据
    Context::set('user_info', ['id' => 1, 'name' => 'name']);
    // 新建协程
    Coroutine::create(function () use ($connection) {
        // 协程间context数据是隔离的，所以新协程里获取的是null
        $userInfo = Context::get('user_info');
        var_dump($userInfo); // 输出null
    });
    // 获取当前协程的context数据
    $userInfo = Context::get('user_info'); // 得到['id' => 1, 'name' => 'name']
    $connection->send(json_encode($userInfo));
};

Worker::runAll();

```

## 接口说明

```php
interface ContextInterface
{
    /**
     * 获取上下文中的值
     */
    public static function get(string $name, mixed $default = null): mixed;

    /**
     * 设置上下文中的值
     */
    public static function set(string $name, mixed $value): void;

    /**
     * 检查上下文中是否存在指定名称的值
     */
    public static function has(string $name): bool;

    /**
     * 重置当前协程上下文
     */
    public static function reset(?ArrayObject $data = null): void;

    /**
     * 销毁上下文
     */
    public static function destroy(): void;

}
```