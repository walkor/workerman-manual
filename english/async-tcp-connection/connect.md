# Connect Method
```php
void AsyncTcpConnection::connect()
```
Performs an asynchronous connection operation. This method returns immediately.

Note: If you need to set the onError callback for asynchronous connection, it should be set before the connect executes, otherwise the onError callback may not be triggered. For example, in the following example, the onError callback may not be triggered and the asynchronous connection failure event may not be captured.

```php
$connection = new AsyncTcpConnection('tcp://baidu.com:81');
// The onError callback is not set when the connection is executed
$connection->connect();
$connection->onError = function($connection, $err_code, $err_msg)
{
    echo "$err_code, $err_msg";
};
```

### Parameters
None

### Return
No return value

### Example with MySQL Proxy

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Actual mysql address, assuming it is on port 3306 on localhost
$REAL_MYSQL_ADDRESS = 'tcp://127.0.0.1:3306';

// Proxy listens on localhost port 4406
$proxy = new Worker('tcp://0.0.0.0:4406');

$proxy->onConnect = function(TcpConnection $connection)
{
    global $REAL_MYSQL_ADDRESS;
    // Asynchronously establish a connection to the actual mysql server
    $connection_to_mysql = new AsyncTcpConnection($REAL_MYSQL_ADDRESS);
    // When the mysql connection receives data, forward it to the corresponding client connection
    $connection_to_mysql->onMessage = function(AsyncTcpConnection $connection_to_mysql, $buffer) use ($connection)
    {
        $connection->send($buffer);
    };
    // When the mysql connection is closed, close the corresponding connection to the client
    $connection_to_mysql->onClose = function(AsyncTcpConnection $connection_to_mysql) use ($connection)
    {
        $connection->close();
    };
    // When an error occurs in the mysql connection, close the corresponding connection to the client
    $connection_to_mysql->onError = function(AsyncTcpConnection $connection_to_mysql) use ($connection)
    {
        $connection->close();
    };
    // Execute asynchronous connection
    $connection_to_mysql->connect();

    // When the client sends data, forward it to the corresponding mysql connection
    $connection->onMessage = function(TcpConnection $connection, $buffer) use ($connection_to_mysql)
    {
        $connection_to_mysql->send($buffer);
    };
    // When the client connection is closed, close the corresponding mysql connection
    $connection->onClose = function(TcpConnection $connection) use ($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };
    // When an error occurs in the client connection, close the corresponding mysql connection
    $connection->onError = function(TcpConnection $connection) use ($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };

};
// Run the worker
Worker::runAll();
```

 **Test**

```sh
mysql -uroot -P4406 -h127.0.0.1 -p
```
Output:
```
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 25004
Server version: 5.5.31-1~dotdeb.0 (Debian)

... (omitted for brevity)
mysql>
```
