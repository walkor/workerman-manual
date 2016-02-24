# GlobalData 组件服务端


# __construct
```php
void \GlobalData\Server::__construct([string $listen_ip = '0.0.0.0', int $listen_port = 2207])
```

实例化一个\GlobalData\Server服务端

### 参数
``` listen_ip ```

监听的本机ip地址，不传默认是```0.0.0.0```

``` listen_port ```

监听的端口，不传默认是2207

## 原理

利用PHP的```__set __get __isset __unset```魔术方法触发与GlobalData服务端通讯，实际变量存储读取等操作都发生在GlobalData服务端。例如当给客户端类设置一个不存在的属性时，会触发```__set```魔术方法，客户端类在```__set```方法中向GlobalData服务端发送请求，存入一个变量。当访问客户端类一个不存在的变量时，会触发类的```__get```方法，客户端会向GlobalData服务端发起请求，读取这个值，从而完成进程间变量共享。
## 例子
```php
use Workerman\Worker;
require_once __DIR__ . '/../../Workerman/Autoloader.php';
require_once __DIR__ . '/../src/Server.php';

$worker = new GlobalData\Server('127.0.0.1', 2207);

Worker::runAll();
```
