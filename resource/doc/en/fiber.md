# Fiber Coroutines
Workerman supports fiber coroutines starting from 5.0.0. 

> **Note**
> Fiber feature requires PHP>=8.1 and installation of `composer require revolt/event-loop ^1.0.0`.

### Introduction
Fiber is a built-in coroutine in PHP, allowing the interruption of PHP code and then resuming its execution when needed. Its main purpose is to allow developers to write asynchronous non-blocking code in a synchronous manner, greatly enhancing code maintainability.

### Example
The following example compares the usage of coroutines and asynchronous callbacks. Suppose we have a requirement to call an HTTP interface and then delay the response by one second. Below are the asynchronous callback and coroutine implementations.

**Asynchronous Callback**
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
    // Request the HTTP interface
    $http->get('http://example.com/', function ($response) use ($connection) {
        // Delay sending by one second
        Timer::add(1, function() use ($connection, $response) {
            // Send data to the browser
            $connection->send((string)$response->getBody());
        }, null, false);
    });
};

Worker::runAll();
```

**Coroutine Usage**
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
    // Call the HTTP interface
    $response = $http->get('http://example.com/');
    // Delay by 1 second
    Timer::sleep(1);
    // Send data
    $connection->send((string)$response->getBody());
};

Worker::runAll();
```

> **Note**
> The above code requires installation of `composer require workerman/http-client ^2.0.0`.

Both approaches execute asynchronously and non-blocking, with good efficiency. However, coroutine usage is easier to read and maintain compared to asynchronous callbacks.

### Fiber Considerations
* Fiber coroutines do not support the coroutinization of Pdo, Redis, and PHP internal blocking functions. This means that using these extensions and functions will still result in blocking calls.
* Currently available Fiber coroutine clients include [workerman/http-client](../components/workerman-http-client.md) and [workerman/redis](../components/workerman-redis.md).

# Swoole Coroutines
Workerman v5 also supports using Swoole as the underlying event driver.

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Swoole\Coroutine\Http\Client;
use Swoole\Coroutine;

// Here we need to manually set Swoole as the underlying event driver
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
**Note**
* It is recommended to use swoole 5.0 or higher versions.
* Setting Swoole as the underlying event driver enables workerman to support Swoole's coroutines.
* When using Swoole as the underlying event driver, it is not necessary to install the event extension.
* By default, Swoole does not have one-click coroutines enabled, meaning that calls based on Pdo, Redis, and PHP built-in file read/write are blocking calls.
* To enable one-click coroutines, manually call `\Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_ALL)`.

For more information, please refer to the [Swoole Manual](https://wiki.swoole.com/).

For further reference, please see [Event Driven](appendices/event.md).

# About Coroutines
First and foremost, there is no need to excessively rely on coroutines. When databases, Redis, and other storage are located within the intranet, multiprocess blocking calls are often faster than coroutines. Based on the benchmark data from [techempower.com over the past 3 years](https://www.techempower.com/benchmarks/#section=data-r21&l=zik073-6bj&test=db), it appears that workerman's blocking database calls perform better than Swoole's database connection pool with coroutines, and even outperform coroutine frameworks such as gin and echo in the Go language by nearly double.

Workerman has already improved the performance of PHP applications by several to tens of times, and in most workerman projects, adding coroutines may not lead to a significant improvement in performance. If there are slow calls in your system, such as external HTTP calls, you can consider using coroutines to improve performance.
