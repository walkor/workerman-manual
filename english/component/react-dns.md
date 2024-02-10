# react/dns
**``` (Requires Workerman version >= 3.3.6) ```**

## Installation:
```composer require react/dns```

## Example:

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;

$worker = new Worker('text://0.0.0.0:6161');

$worker->onWorkerStart = function() {
    global   $dns;
    $loop    = Worker::getEventLoop();
    $factory = new React\Dns\Resolver\Factory();
    $dns     = $factory->create('8.8.8.8', $loop);
};

$worker->onMessage = function($connection, $host) {
    global $dns;
    $dns->resolve($host)->then(function($ip) use($host, $connection) {
        $connection->send("$host: $ip");
    }, function($e) use($host, $connection) {
        $connection->send("$host: {$e->getMessage()}");
    });
};

Worker::runAll();
```

## Documentation:
https://github.com/reactphp/dns

## Note:
1. All asynchronous code must be written within the `onXXX` callback.

2. Use `Worker::getEventLoop();` to obtain the `$loop` variable required by asynchronous clients.
