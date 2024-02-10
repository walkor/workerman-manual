＃ トランスポート
## 説明：
```php
string Worker::$transport
```

現在のWorkerインスタンスが使用するトランスポート層プロトコルを設定します。現在、3種類のみサポートしています（tcp、udp、ssl）。デフォルトでは設定されていない場合はtcpです。

```注意：sslにはWorkermanバージョン>=3.3.7が必要です```


## 例1

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:8484');
// UDPプロトコルを使用
$worker->transport = 'udp';
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('Hello');
};
// Workerを実行
Worker::runAll();
```


## 例2

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 証明書はできるだけ申請された証明書を使用することが望ましいです
$context = array(
    'ssl' => array(
        'local_cert' => '/etc/nginx/conf.d/ssl/server.pem', // crtファイルでも構いません
        'local_pk'   => '/etc/nginx/conf.d/ssl/server.key',
    )
);
// ここでwebsocketプロトコルを設定します
$worker = new Worker('websocket://0.0.0.0:4431', $context);
// transportをsslに設定し、websocket+sslはwssです
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```
