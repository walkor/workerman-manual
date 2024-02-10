# connect メソッド
```php
void AsyncTcpConnection::connect()
```
非同期の接続操作を実行します。このメソッドはすぐにリターンします。

注意：非同期接続のonErrorコールバックを設定する必要がある場合は、connectを実行する前に設定する必要があります。そうしないと、例えば以下の例のようにonErrorコールバックがトリガーされず、非同期接続の失敗イベントをキャッチできない可能性があります。

```php
$connection = new AsyncTcpConnection('tcp://baidu.com:81');
// connectが実行される前にonErrorコールバックが設定されていない
$connection->connect();
$connection->onError = function($connection, $err_code, $err_msg)
{
    echo "$err_code, $err_msg";
};
```

### パラメーター
パラメーターなし

### 戻り値
戻り値なし

### サンプル　MySQLプロキシ

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 実際のMySQLアドレス、ここではローカルの3306ポートと仮定
$REAL_MYSQL_ADDRESS = 'tcp://127.0.0.1:3306';

// 代理はローカルの4406ポートをリッスン
$proxy = new Worker('tcp://0.0.0.0:4406');

$proxy->onConnect = function(TcpConnection $connection)
{
    global $REAL_MYSQL_ADDRESS;
    // 実際のMySQLサーバへの非同期接続を確立
    $connection_to_mysql = new AsyncTcpConnection($REAL_MYSQL_ADDRESS);
    // MySQL接続からデータを受信した場合、対応するクライアント接続に転送する
    $connection_to_mysql->onMessage = function(AsyncTcpConnection $connection_to_mysql, $buffer) use ($connection)
    {
        $connection->send($buffer);
    };
    // MySQL接続が閉じられた場合、対応するクライアントへの接続も閉じる
    $connection_to_mysql->onClose = function(AsyncTcpConnection $connection_to_mysql) use ($connection)
    {
        $connection->close();
    };
    // MySQL接続でエラーが発生した場合、対応するクライアントへの接続も閉じる
    $connection_to_mysql->onError = function(AsyncTcpConnection $connection_to_mysql) use ($connection)
    {
        $connection->close();
    };
    // 非同期接続を実行
    $connection_to_mysql->connect();

    // クライアントからデータを受信した場合、対応するMySQL接続に転送する
    $connection->onMessage = function(TcpConnection $connection, $buffer) use ($connection_to_mysql)
    {
        $connection_to_mysql->send($buffer);
    };
    // クライアント接続が切断された場合、対応するMySQL接続も閉じる
    $connection->onClose = function(TcpConnection $connection) use ($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };
    // クライアント接続でエラーが発生した場合、対応するMySQL接続も閉じる
    $connection->onError = function(TcpConnection $connection) use ($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };
};
// workerを実行
Worker::runAll();
```

**テスト**

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
