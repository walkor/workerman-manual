# connect 方法
```php
void AsyncTcpConnection::connect()
```
执行异步连接操作。此方法会立刻返回。

注意：如果需要设置异步连接的onError回调，则应该在connect执行之前设置，否则onError回调可能无法被触发，例如下面的例子onError回调可能无法触发，无法捕获异步连接失败事件。

```php
$connection = new AsyncTcpConnection('tcp://baidu.com:81');
// 执行连接的时候还没设置onError回调
$connection->connect();
$connection->onError = function($connection, $err_code, $err_msg)
{
    echo "$err_code, $err_msg";
};
```

### 参数
无参数


### 返回值
无返回值

### 示例 Mysql代理

```php
use \Workerman\Worker;
use \Workerman\Connection\AsyncTcpConnection;
require_once './Workerman/Autoloader.php';

// 真实的mysql地址，假设这里是本机3306端口
$REAL_MYSQL_ADDRESS = 'tcp://127.0.0.1:3306';

// 代理监听本地4406端口
$proxy = new Worker('tcp://0.0.0.0:4406');

$proxy->onConnect = function($connection)
{
    global $REAL_MYSQL_ADDRESS;
    // 异步建立一个到实际mysql服务器的连接
    $connection_to_mysql = new AsyncTcpConnection($REAL_MYSQL_ADDRESS);
    // mysql连接发来数据时，转发给对应客户端的连接
    $connection_to_mysql->onMessage = function($connection_to_mysql, $buffer)use($connection)
    {
        $connection->send($buffer);
    };
    // mysql连接关闭时，关闭对应的代理到客户端的连接
    $connection_to_mysql->onClose = function($connection_to_mysql)use($connection)
    {
        $connection->close();
    };
    // mysql连接上发生错误时，关闭对应的代理到客户端的连接
    $connection_to_mysql->onError = function($connection_to_mysql)use($connection)
    {
        $connection->close();
    };
    // 执行异步连接
    $connection_to_mysql->connect();

    // 客户端发来数据时，转发给对应的mysql连接
    $connection->onMessage = function($connection, $buffer)use($connection_to_mysql)
    {
        $connection_to_mysql->send($buffer);
    };
    // 客户端连接断开时，断开对应的mysql连接
    $connection->onClose = function($connection)use($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };
    // 客户端连接发生错误时，断开对应的mysql连接
    $connection->onError = function($connection)use($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };

};
// 运行worker
Worker::runAll();
```

 **测试**

```shell
mysql -uroot -P4406 -h127.0.0.1 -p

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 25004
Server version: 5.5.31-1~dotdeb.0 (Debian)

Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>

 ```

