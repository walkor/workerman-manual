# 基本调试

WorkerMan3.0有两种运行模式，调试模式以及daemon运行模式

运行 ```php start.php start``` 进入调试模式，这时代码中的```echo、var_dump、var_export```等函数打印会在终端显示。注意以```php start.php start```运行的WorkerMan在终端关闭时所有进程会退出。

而运行 ```php start.php start -d```则是进入daemon模式，也就是正式上线的运行模式，关闭终端不受影响。


如果想daemon方式运行时也能看到```echo、var_dump、var_export```等函数打印，可以设置Worker::$stdoutFile属性，例如

```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

// 将屏幕打印输出到Worker::$stdoutFile指定的文件中
Worker::$stdoutFile = '/tmp/stdout.log';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function($connection, $data)
{
    $connection->send('hello world');
};
```
这样所有的```echo、var_dump、var_export```等函数打印会写入到```Worker::$stdoutFile```指定的文件中。注意```Worker::$stdoutFile```指定的路径要有可写权限。


