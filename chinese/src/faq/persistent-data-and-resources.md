# 对象和资源的持久化
在传统的Web开发中，PHP创建的对象、数据、资源等会在请求完毕后全部释放，导致很难做到持久化。而在WorkerMan中可以轻松做到这些。

在WorkerMan中如果想在内存中永久保存某些数据资源，可以将资源放到全局变量中或者类的静态成员中。


例如下面的代码：

用一个全局变量```$connection_count```保存一个当前进程的客户端连接数。

```php
<?php
use \Workerman\Worker;
require_once './Workerman/Autoloader.php';

// 全局变量，保存当前进程的客户端连接数
$connection_count = 0;

$worker = new Worker('tcp://0.0.0.0:1236');

$worker->onConnect = function($connection)
{
    // 有新的客户端连接时，连接数+1
    global $connection_count;
    ++$connection_count;
    echo "now connection_count=$connection_count\n";
};

$worker->onClose = function($connection)
{
    // 客户端关闭时，连接数-1
    global $connection_count;
    $connection_count--;
    echo "now connection_count=$connection_count\n";
};

```


##PHP变量作用域参见：
http://php.net/manual/zh/language.variables.scope.php


