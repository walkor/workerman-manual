# Events Supported by Workerman

| Name  | Dependent Extension | Coroutine Supported | Priority | Workerman Version |
|-----|------|--|-----|
|  Workerman\Events\Select   |   None   | Not supported  |  Kernel default support   |  >=3.0  ｜
|  Workerman\Events\Revolt   |   event (optional)   | Supported |  Requires [revolt/event-loop](https://github.com/revoltphp/event-loop)   |  >=5.0  |
|  Workerman\Events\Event   |   event   | Not supported |  Kernel default support   |  >=3.0  |
|  Workerman\Events\Swoole   |  [swoole](https://github.com/swoole/swoole-src)   | Supported |  Requires manual setting   |  >=4.0  |
|  Workerman\Events\Swow   |   [swow](https://github.com/swow/swow)   | Supported |  Requires manual setting   |  >=5.0  |

* Each kernel driver provides unique features, for example, using `Revolt` will enable Workerman to support PHP's built-in [Fiber coroutine](https://www.php.net/manual/zh/language.fibers.php), while using `Swoole` will enable Workerman to support Swoole's coroutine.
* The event drivers are mutually exclusive. For example, when using `Revolt`'s Fiber coroutine, Swoole or Swow's coroutine cannot be used.
* `Revolt` requires the installation of `composer require revolt/event-loop ^1.0.0`, and after installation, the Workerman kernel will automatically set it as the preferred event driver.
* `Swoole` and `Swow` need to be manually set using `Worker::$eventLoopClass` to take effect (refer to the next paragraph).
* By default, Swoole does not enable [Coroutine Runtime](https://wiki.swoole.com/#/runtime?id=runtime), which means that calls based on Pdo, Redis, and PHP's built-in file read/write are still blocking.
* To enable Swoole's Coroutine Runtime, you need to manually call `\Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_ALL);`.

> **Note**
> The Swow extension will automatically change the behavior of some built-in PHP functions, which may cause Workerman to be unresponsive to requests and signals if Swow is enabled but not used as the event driver. Therefore, if you do not use Swow as the underlying driver, you need to comment out Swow from php.ini.

For more information, refer to [Workerman Fiber](../fiber.md).

# Setting Event Driver for Workerman

Below is the manual setting of the event driver for Workerman:

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Swoole\Coroutine\Http\Client;
use Swoole\Coroutine;

// Manually set as the underlying event driver
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
