# sleep
```php
int \Workerman\Timer::sleep(float $delay)
```
類似於內建的 `sleep()` 函數，不同之處在於 `Timer::sleep()` 是非阻塞的（不會阻塞當前進程）。

> **注意**
> 此功能需要workerman>=5.0
> 此功能需要安裝 composer require revolt/event-loop ^1.0.0 ，或者使用Swoole/Swow作為事件驅動

### 參數
``` delay ```

多長時間執行一次，單位秒，支持小數，可以精確到0.001，即精確到毫秒級別。

### 返回值
無返回值

### 示例

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    // 延遲1.5秒發送數據
    Timer::sleep(1.5);
    // 發送數據
    $connection->send('hello workerman');
};

Worker::runAll();
```
