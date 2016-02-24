# GlobalData 组件客户端


# __construct
```php
void \GlobalData\Client::__construct([string $global_server_ip = '0.0.0.0', int $global_server_port = 2207])
```

实例化一个\GlobalData\Client客户端

### 参数
``` global_server_ip ```

监听的本机ip地址，不传默认是```0.0.0.0```

``` global_server_port ```

监听的端口，不传默认是2207

## 说明
支持赋值、读取、isset、unset操作。
同时支持cas原子操作。


## 例子


```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/../Workerman/Autoloader.php';
require_once __DIR__ . '/client.php';

$worker = new Worker('tcp://0.0.0.0:6636');
// 进程启动时
$worker->onWorkerStart = function(){
    // 初始化一个全局的global data client
    global $global;
    $global = new Client();
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

### 全部用法
```php
require_once __DIR__ . '/../src/Client.php';

$global = new GlobalData\Client('127.0.0.1', 2207);

// isset判断$global->somedata是否设置
var_export(isset($global->somedata));
// 设置$global->somedata的值，其它进程会共享这个值
$global->somedata = array(1,2,3);
// 打印当前值
var_export($global->somedata);
// 删除$global->somedata
unset($global->somedata);
// 打印
var_export($global->somedata);
// 重新赋值
$global->somedata = array(1,2,3);
// 打印
var_export($global->somedata);
// 原子替换
var_export($global->cas('somedata', array(1,2,3), array(5,6,7)));
// 打印
var_export($global->somedata);
```

