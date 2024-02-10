# Fiber协程
workerman从5.0.0开始支持[Fiber协程(纤程)](https://www.php.net/manual/zh/language.fibers.php)

> **注意**
> Fiber特性需要PHP>=8.1并安装 `composer require revolt/event-loop ^1.0.0`

### 介绍

Fiber是php内置的协程(纤程)，它可以中断PHP代码，然后在需要的时候恢复其运行。它的最大作用是让开发者可以用同步的方式写异步非阻塞代码，这极大地增强了代码的可维护性。

### 示例
下面通过示例来对比协程和异步回调编程的区别。
假设我们有一个需求要求需要调用一个HTTP接口，然后延迟一秒响应，异步回调及协程写法分别如下。

**异步回调用法**
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
    // 请求HTTP接口
    $http->get('http://example.com/', function ($response) use ($connection) {
        // 延迟一秒发送
        Timer::add(1, function() use ($connection, $response) {
            // 向浏览器发送数据
            $connection->send((string)$response->getBody());
        }, null, false);
    });
};

Worker::runAll();
```

**协程用法**
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
    // 调用HTTP接口
    $response = $http->get('http://example.com/');
    // 延迟1秒
    Timer::sleep(1);
    // 发送数据
    $connection->send((string)$response->getBody());
};

Worker::runAll();
```

> **注意**
> 以上代码需要安装 composer require workerman/http-client ^2.0.0

这两种写法都是异步非阻塞执行的，运行效率都很好，但是协程用法比异步回调更容易阅读，更容易维护。


### Fiber注意事项
* Fiber协程并不支持将Pdo、Redis、及PHP内部阻塞函数协程化，也就是说使用这些扩展及函数仍然是阻塞式调用
* 目前可用的Fiber协程客户端有 [workerman/http-client](../components/workerman-http-client.md)，[workerman/redis](../components/workerman-redis.md)

# Swoole协程
workerman v5 同时支持将Swoole作为底层事件驱动


```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Swoole\Coroutine\Http\Client;
use Swoole\Coroutine;

// 这里需要手动将Swoole设置为底层事件驱动
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
**提示**
* 建议使用 swoole5.0 或者后续更高版本
* 把swoole作为底层事件驱动可以让workerman支持swoole的协程
* 使用swoole作为底层事件驱动时可以不安装event扩展
* swoole默认没有开启一键协程，也就是说基于Pdo、Redis、PHP内置文件读写是阻塞式调用
* 如需开启一键协程，需要手动调用 `\Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_ALL);`

更多请参考[Swoole手册](https://wiki.swoole.com/)

更多请参考[事件驱动](appendices/event.md)

# 关于协程
首先没必过份迷信协程，当数据库、Redis等存储都在内网时，多进程阻塞式调用很多时候比协程更快。从[techempower.com 3年来的压测数据](https://www.techempower.com/benchmarks/#section=data-r21&l=zik073-6bj&test=db)来看workerman阻塞式数据库调用性能要优于swoole数据库连接池+协程，甚至比go语言的gin、echo等协程框架性能高近1倍。

workerman已经将PHP应用的性能提升了数倍甚至数十倍，绝大部分workerman项目加上协程可能不会有更大的性能提升。
如果你的系统里有慢调用，例如有外部HTTP调用，可以考虑使用协程来提升性能。

