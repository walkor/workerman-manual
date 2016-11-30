# react/child-process

## 安装：
```
composer require react/child-process
```

## 示例：

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;

$worker = new Worker();


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

## 文档：
https://github.com/reactphp/child-process

## 注意：
1、所有的异步编码必须在```onXXX```回调中编写

2、异步客户端需要的```$loop```变量请使用```Worker::getEventLoop();```返回值





