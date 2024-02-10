# connect 方法
```php
void AsyncTcpConnection::connect()
```
執行異步連接操作。該方法將立即返回。

注意：如需設置異步連接的onError回調，應該在connect執行之前設置，否則onError回調可能無法被觸發，例如下面的例子onError回調可能無法觸發，無法捕獲異步連接失敗事件。

```php
$connection = new AsyncTcpConnection('tcp://baidu.com:81');
// 執行連接的時候還沒設置onError回調
$connection->connect();
$connection->onError = function($connection, $err_code, $err_msg)
{
    echo "$err_code, $err_msg";
};
```

### 參數
無參數


### 返回值
無返回值

### 示例 Mysql代理

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 真實的mysql地址，假設這裡是本機3306端口
$REAL_MYSQL_ADDRESS = 'tcp://127.0.0.1:3306';

// 代理監聽本地4406端口
$proxy = new Worker('tcp://0.0.0.0:4406');

$proxy->onConnect = function(TcpConnection $connection)
{
    global $REAL_MYSQL_ADDRESS;
    // 異步建立一個到實際mysql伺服器的連接
    $connection_to_mysql = new AsyncTcpConnection($REAL_MYSQL_ADDRESS);
    // mysql連接發來數據時，轉發給對應客戶端的連接
    $connection_to_mysql->onMessage = function(AsyncTcpConnection $connection_to_mysql, $buffer)use($connection)
    {
        $connection->send($buffer);
    };
    // mysql連接關閉時，關閉對應的代理到客戶端的連接
    $connection_to_mysql->onClose = function(AsyncTcpConnection $connection_to_mysql)use($connection)
    {
        $connection->close();
    };
    // mysql連接上發生錯誤時，關閉對應的代理到客戶端的連接
    $connection_to_mysql->onError = function(AsyncTcpConnection $connection_to_mysql)use($connection)
    {
        $connection->close();
    };
    // 執行異步連接
    $connection_to_mysql->connect();

    // 客戶端發來數據時，轉發給對應的mysql連接
    $connection->onMessage = function(TcpConnection $connection, $buffer)use($connection_to_mysql)
    {
        $connection_to_mysql->send($buffer);
    };
    // 客戶端連接斷開時，斷開對應的mysql連接
    $connection->onClose = function(TcpConnection $connection)use($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };
    // 客戶端連接發生錯誤時，斷開對應的mysql連接
    $connection->onError = function(TcpConnection $connection)use($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };

};
// 運行worker
Worker::runAll();
```

 **測試**

```
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
