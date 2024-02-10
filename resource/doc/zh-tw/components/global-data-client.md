# GlobalData 組件客戶端
**``` (要求Workerman版本>=3.3.0) ```**

# __construct
```php
void \GlobalData\Client::__construct(mixed $server_address)
```

實例化一個\GlobalData\Client客戶端物件。透過在客戶端物件上賦值屬性來進程間共享數據。

### 參數
GlobalData server 服務端地址，格式```<ip地址>:<端口>```，例如```127.0.0.1:2207```。

如果是GlobalData server集群，則傳入一個地址數組，例如```array('10.0.0.10:2207', '10.0.0.0.11:2207')```

## 說明
支持賦值、讀取、isset、unset操作。

同時支持cas原子操作。


## 例子

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// GlobalData Server
$global_worker = new GlobalData\Server('0.0.0.0', 2207);

$worker = new Worker('tcp://0.0.0.0:6636');
// 進程啟動時
$worker->onWorkerStart = function()
{
    // 初始化一個全局的global data client
    global $global;
    $global = new \GlobalData\Client('127.0.0.1:2207');
};
// 每次服務端收到消息時
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // 更改$global->somedata的值，其它進程會共享這個$global->somedata變量
    global $global;
    echo "now global->somedata=".var_export($global->somedata, true)."\n";
    echo "set \$global->somedata=$data";
    $global->somedata = $data;
};
Worker::runAll();
```

### 全部用法（php-fpm環境也可以使用）
```php
require_once __DIR__ . '/vendor/autoload.php';

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
GlobalData組件無法共享資源類型的數據，例如mysql連接、socket連接等無法共享。

如果在Workerman環境中使用GlobalData/Client，請在onXXX回調中實例化GlobalData/Client物件，例如在onWorkerStart中實例化。

不能這樣操作共享變量。
```php
$global->somekey = array();
$global->somekey[]='xxx';

$global->someObject = new someClass();
$global->someObject->someVar = 'xxx';
```
可以這樣
```php
$somekey = array();
$somekey[] = 'xxx';
$global->somekey = $somekey;

$someObject = new someClass();
$someObject->someVar = 'xxx';
$global->someObject = $someObject;
```
