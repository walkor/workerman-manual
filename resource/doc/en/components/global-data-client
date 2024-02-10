# GlobalData 组件客户端
**``` (要求Workerman版本>=3.3.0) ```**

# __construct
```php
void \GlobalData\Client::__construct(mixed $server_address)
```

实例化一个\GlobalData\Client客户端对象。通过在客户端对象上赋值属性来进程间共享数据。

### 参数
GlobalData server 服务端地址，格式```<ip地址>:<端口>```，例如```127.0.0.1:2207```。

如果是GlobalData server集群，则传入一个地址数组，例如```array('10.0.0.10:2207', '10.0.0.0.11:2207')```

## 说明
支持赋值、读取、isset、unset操作。
同时支持cas原子操作。


## 例子

```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/Workerman/Autoloader.php';
require_once __DIR__ . '/GlobalData/src/Client.php';

$worker = new Worker('tcp://0.0.0.0:6636');
// 进程启动时
$worker->onWorkerStart = function()
{
    // 初始化一个全局的global data client
    global $global;
    $global = new \GlobalData\Client('127.0.0.1:2207');
};
// 每次服务端收到消息时
$worker->onMessage = function($connection, $data)
{
    // 更改$global->somedata的值，其它进程会共享这个$global->somedata变量
    global $global;
    echo "now global->somedata=".var_export($global->somedata, true)."\n";
    echo "set \$global->somedata=$data";
    $global->somedata = $data;
};
Worker::runAll();
```

### 全部用法（php-fpm环境也可以使用）
```php
require_once __DIR__ . '/GlobalData/src/Client.php';

$global = new Client('127.0.0.1:2207');

var_export(isset($global->abc));

$global->abc = array(1,2,3);

var_export($global->abc);

unset($global->abc);

var_export($global->add('abc', 10));

var_export($global->increment('abc', 2));

var_export($global->cas('abc', 12, 18));

```

## 注意：
GlobalData组件无法共享资源类型的数据，例如mysql连接、socket连接等无法共享。

如果在Workerman环境中使用GlobalData/Client，请在onXXX回调中实例化GlobalData/Client对象，例如在onWorkerStart中实例化。

不能这样操作共享变量。
```php
$global->somekey = array();
$global->somekey[]='xxx';

$global->someObject = new someClass();
$global->someObject->someVar = 'xxx';
```
可以这样
```php
$somekey = array();
$somekey[] = 'xxx';
$global->somekey = $somekey;

$someObject = new someClass();
$someObject->someVar = 'xxx';
$global->someObject = $someObject;
```
