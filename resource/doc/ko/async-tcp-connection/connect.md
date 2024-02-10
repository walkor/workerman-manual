```php
void AsyncTcpConnection::connect()
```
비동기 연결 작업을 실행합니다. 이 메서드는 즉시 반환됩니다.

참고: onError 콜백을 설정해야 하는 경우에는 connect를 실행하기 전에 설정해야 합니다. 그렇지 않으면 onError 콜백이 트리거되지 않을 수 있습니다. 예를 들어 아래의 예에서는 onError 콜백이 트리거되지 않아 비동기 연결 실패 이벤트를 캡처할 수 없습니다.

```php
$connection = new AsyncTcpConnection('tcp://baidu.com:81');
// 연결 실행 시에 onError 콜백을 아직 설정하지 않음
$connection->connect();
$connection->onError = function($connection, $err_code, $err_msg)
{
    echo "$err_code, $err_msg";
};
```

### 매개변수
매개변수 없음

### 반환값
반환값 없음

### 예시 Mysql 프록시

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 실제 mysql 주소, 여기서는 예를 들어 로컬 3306 포트입니다.
$REAL_MYSQL_ADDRESS = 'tcp://127.0.0.1:3306';

// 프록시는 로컬 4406 포트를 수신합니다.
$proxy = new Worker('tcp://0.0.0.0:4406');

$proxy->onConnect = function(TcpConnection $connection)
{
    global $REAL_MYSQL_ADDRESS;
    // 실제 mysql 서버에 대한 비동기 연결 설정
    $connection_to_mysql = new AsyncTcpConnection($REAL_MYSQL_ADDRESS);
    // mysql 연결로부터 데이터가 도착하면 해당 클라이언트 연결로 전달
    $connection_to_mysql->onMessage = function(AsyncTcpConnection $connection_to_mysql, $buffer) use ($connection)
    {
        $connection->send($buffer);
    };
    // mysql 연결이 닫힐 때 해당 대리자에서 클라이언트 연결을 닫음
    $connection_to_mysql->onClose = function(AsyncTcpConnection $connection_to_mysql) use ($connection)
    {
        $connection->close();
    };
    // mysql 연결에서 오류가 발생할 때 해당 대리자에서 클라이언트 연결을 닫음
    $connection_to_mysql->onError = function(AsyncTcpConnection $connection_to_mysql) use ($connection)
    {
        $connection->close();
    };
    // 비동기 연결 실행
    $connection_to_mysql->connect();

    // 클라이언트로부터 데이터가 도착하면 해당 mysql 연결로 전달
    $connection->onMessage = function(TcpConnection $connection, $buffer) use ($connection_to_mysql)
    {
        $connection_to_mysql->send($buffer);
    };
    // 클라이언트 연결이 닫힐 때 해당 mysql 연결도 닫음
    $connection->onClose = function(TcpConnection $connection) use ($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };
    // 클라이언트 연결에서 오류가 발생할 때 해당 mysql 연결도 닫음
    $connection->onError = function(TcpConnection $connection) use ($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };

};
// worker 실행
Worker::runAll();
```

 **테스트**

```bash
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
