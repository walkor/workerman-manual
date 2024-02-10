# react/child-process
**``` (Requires Workerman version >= 3.3.6) ```**

## Installation:
```php
composer require react/child-process
```

## Example:

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;

$worker = new Worker('tcp://127.0.0.1:1234');

$worker->onWorkerStart = function() {
    $loop    = Worker::getEventLoop();

    $process = new React\ChildProcess\Process('echo hello');

    $process->start($loop);

    $process->on('exit', function($exitCode, $termSignal) {
        echo "cmd complete\n";
    });

    $process->stdout->on('data', function($output) {
        echo $output;
    });

    $process->stderr->on('data', function($output) {
        echo $output;
    });
};

Worker::runAll(); 
```

## Documentation:
https://github.com/reactphp/child-process

## Note:
1. All asynchronous code must be written in the `onXXX` callback.
2. Use the `$loop` variable required by asynchronous clients by using the return value of `Worker::getEventLoop();`.
