# runAll
```php
void Worker::runAll(void)
```
运行所有Worker实例。

**注意：**

Worker::runAll()执行后将永久阻塞，也就是说位于Worker::runAll()后面的代码将不会被执行。所有Worker实例化应该都在Worker::runAll()前进行。

### 参数
无参数



### 返回值
无返回

## 范例 运行多个Worker实例

start.php

```php
<?php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function($connection, $data)
{
    $connection->send('hello http');
};

$ws_worker = new Worker('websocket://0.0.0.0:4567');
$ws_worker->onMessage = function($connection, $data)
{
    $connection->send('hello websocket');
};

// 运行所有Worker实例
Worker::runAll();
```


**注意：**

windows版本的workerman不支持在同一个文件中实例化多个Worker。
上面的例子无法在windows版本的workerman下运行。

windows版本的workerman需要将多个Worker实例初始化放在不同的文件中，像下面这样

start_http.php


```php
<?php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function($connection, $data)
{
    $connection->send('hello http');
};

// 运行所有Worker实例(这里只有一个实例)
Worker::runAll();
```

start_websocket.php


```php
<?php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$ws_worker = new Worker('websocket://0.0.0.0:4567');
$ws_worker->onMessage = function($connection, $data)
{
    $connection->send('hello websocket');
};

// 运行所有Worker实例(这里只有一个实例)
Worker::runAll();
```




