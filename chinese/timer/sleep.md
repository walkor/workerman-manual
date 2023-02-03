# sleep
```php
int \Workerman\Timer::sleep(float $delay)
```
类似PHP内置的`sleep()`函数，区别是`Timer::sleep()`是非阻塞的(不会阻塞当前进程)

> **注意**
> 此特性需要workerman>=5.0
> 此特性需要安装 composer require revolt/event-loop ^1.0.0 ，或者使用Swoole/Swow作为事件驱动


### 参数
 ``` delay ```

多长时间执行一次，单位秒，支持小数，可以精确到0.001，即精确到毫秒级别。

### 返回值
无返回值

### 示例

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    // 延迟1.5秒发送数据
    Timer::sleep(1.5);
    // 发送数据
    $connection->send('hello workerman');
};

Worker::runAll();
```


