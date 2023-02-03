# workerman目前支持的事件驱动

| 名称  | 依赖扩展 | 支持协程 |  优先级  |  workerman版本  |
|-----|------|--|-----|
|  Workerman\Events\Select   |   无   | 不支持  |  内核默认支持   |  >=3.0  ｜
|  Workerman\Events\Revolt   |   event(可选)   | 支持 |  需安装[revolt/event-loop](https://github.com/revoltphp/event-loop)   |  >=5.0  |
|  Workerman\Events\Event   |   event   | 不支持 |  内核默认支持   |  >=3.0  |
|  Workerman\Events\Swoole   |  [swoole](https://github.com/swoole/swoole-src)   | 支持 |  需手动设置   |  >=4.0  |
|  Workerman\Events\Swow   |   [swow](https://github.com/swow/swow)   | 支持 |  需手动设置   |  >=5.0  |

* 每种内核驱动会提供单独的特性，例如使用`Revolt`会让workerman支持PHP内置的[Fiber协程(纤程)](https://www.php.net/manual/zh/language.fibers.php)，使用`Swoole`则会让workerman支持Swoole的协程
* 各个事件驱动之间是互斥关系，例如使用`Revolt`的Fiber协程时，无法使用Swoole或者Swow的协程
* `Revolt`需要安装`composer require revolt/event-loop ^1.0.0`，安装后workerman内核自动将其设置为首选事件驱动
* `Swoole` 和 `Swow`需要手动设置`Worker::$eventLoopClass`才能生效(参见下一段落)
* swoole默认没有开启[一键协程Runtime](https://wiki.swoole.com/#/runtime?id=runtime)，也就是说基于Pdo、Redis、PHP内置文件读写仍是阻塞式调用
* 如需swoole开启一键协程，需要手动调用 `\Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_ALL);`

> **注意**
> swow扩展会自动更改一些PHP内置函数的行为，这会导致开启了swow扩展但是没有使用swow作为事件驱动时，workerman无法响应请求以及信号，所以如果您不使用swow作为底层驱动时，需要将swow从php.ini中注释掉

更多参考[workerman协程](../fiber.md)

# 为workerman设置事件驱动

以下是为workerman手动设置事件驱动

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Swoole\Coroutine\Http\Client;
use Swoole\Coroutine;

// 手动设置为底层事件驱动
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
