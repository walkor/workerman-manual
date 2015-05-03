# connect
## Description:
```php
void \Workerman\Connection\AsyncTcpConnection::connect()
```
Opens the connection.

### Parameters
This function has no parameters.


### Return Values
No value is returned.

### Examples: Mysql proxy

```php
use \Workerman\Worker;
use \Workerman\Connection\AsyncTcpConnection;

$REAL_MYSQL_ADDRESS = 'tcp://127.0.0.1:3306';

$proxy = new Worker('tcp://0.0.0.0:4406');

$proxy->onConnect = function($connection)
{
    global $REAL_MYSQL_ADDRESS;

    $connection_to_mysql = new AsyncTcpConnection($REAL_MYSQL_ADDRESS);

    $connection_to_mysql->onMessage = function($connection_to_mysql, $buffer)use($connection)
    {
        $connection->send($buffer);
    };

    $connection_to_mysql->onClose = function($connection_to_mysql)use($connection)
    {
        $connection->close();
    };

    $connection_to_mysql->onError = function($connection_to_mysql)use($connection)
    {
        $connection->close();
    };

    $connection_to_mysql->connect();

    $connection->onMessage = function($connection, $buffer)use($connection_to_mysql)
    {
        $connection_to_mysql->send($buffer);
    };

    $connection->onClose = function($connection)use($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };

    $connection->onError = function($connection)use($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };

};
```

 **Test**

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

