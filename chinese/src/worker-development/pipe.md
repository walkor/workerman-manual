# pipe
## 说明:
```php
void Connection::pipe(TcpConnection $target_connection)
```



## 参数
将当前连接的数据流导入到目标连接。内置了流量控制。此方法做TCP代理非常有用



## 范例 TCP代理

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once './Workerman/Autoloader.php';

$worker = new Worker('tcp://0.0.0.0:8483');
$worker->count = 12;

// tcp连接建立后
$worker->onConnect = function($connection)
{
    // 建立本地80端口的异步连接
    $connection_to_80 = new AsyncTcpConnection('tcp://127.0.0.1:80');
    // 设置将当前客户端连接的数据导向80端口的连接
    $connection->pipe($connection_to_80);
    // 设置80端口连接返回的数据导向客户端连接
    $connection_to_80->pipe($connection);
    // 执行异步连接
    $connection_to_80->connect();
};

// 运行worker
Worker::runAll();
```

