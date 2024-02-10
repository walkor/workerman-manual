# 傳輸層
## 說明:
```php
string Worker::$transport
```

設置當前Worker實例所使用的傳輸層協議，目前只支持3種(tcp、udp、ssl)。若未設置，默認為tcp。

``` 注意：ssl需要Workerman版本>=3.3.7 ```

## 範例 1
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:8484');
// 使用udp協議
$worker->transport = 'udp';
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('Hello');
};
// 運行worker
Worker::runAll();
```

## 範例 2
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 憑證最好是申請的憑證
$context = array(
    'ssl' => array(
        'local_cert' => '/etc/nginx/conf.d/ssl/server.pem', // 也可以是crt文件
        'local_pk'   => '/etc/nginx/conf.d/ssl/server.key',
    )
);
// 這裡設置的是websocket協議
$worker = new Worker('websocket://0.0.0.0:4431', $context);
// 設置transport開啟ssl，websocket+ssl即wss
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```
