# react/mysql

**``` (要求Workerman版本>=3.3.6) ```**

## 安装：
```
composer require react/mysql
```

## 示例：

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;

$worker = new Worker('text://0.0.0.0:6161');

// 进程启动时
$worker->onWorkerStart = function() {
    global $mysql;
    // 获得workerman的event-loop，
    $loop  = Worker::getEventLoop();
    // 连接参数
    $mysql = new React\MySQL\Connection($loop, array(
        'host'   => '127.0.0.1', // 不要写localhost
        'dbname' => '数据库名',
        'user'   => '用户名',
        'passwd' => '密码',
    ));
    // 出现错误时
    $mysql->on('error', function($e){
        echo $e;
    });
    // 执行连接
    $mysql->connect(function ($e) {
        if($e) {
            echo $e;
        } else {
            echo "connect success\n";
        }
    });
};
// 收到客户端请求时
$worker->onMessage = function($connection, $data) {
    global $mysql;
    // 执行异步查询
    $mysql->query('show databases' /*$data*/, function ($command, $mysql) use ($connection) {
        if ($command->hasError()) {
            $error = $command->getError();
        } else {
            $results = $command->resultRows;
            $fields  = $command->resultFields;
            $connection->send(json_encode($results));
        }
    });
};

Worker::runAll();
```

## 文档：
https://github.com/bixuehujin/reactphp-mysql

## 注意：
1、所有的异步编码必须在```onXXX```回调中编写

2、异步客户端需要的```$loop```变量请使用```Worker::getEventLoop();```返回值


