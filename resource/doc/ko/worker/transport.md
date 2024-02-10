# 전송
## 설명:
```php
string Worker::$transport
```

현재 Worker 인스턴스에서 사용하는 전송 프로토콜을 설정합니다. 현재는 3가지 유형(tcp, udp, ssl)만 지원됩니다. 설정하지 않으면 기본값은 tcp입니다.

``` 주의: ssl은 Workerman 버전 >=3.3.7이 필요합니다. ```

## 예제 1

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:8484');
// udp 프로토콜 사용
$worker->transport = 'udp';
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('Hello');
};
// worker 실행
Worker::runAll();
```

## 예제 2

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 인증서는 베스트프랙티스로 설치해야 합니다.
$context = array(
    'ssl' => array(
        'local_cert' => '/etc/nginx/conf.d/ssl/server.pem', // crt 파일도 가능합니다.
        'local_pk'   => '/etc/nginx/conf.d/ssl/server.key',
    )
);
// 여기서는 웹소켓 프로토콜을 설정합니다.
$worker = new Worker('websocket://0.0.0.0:4431', $context);
// transport를 통해 ssl 활성화, websocket+ssl인 wss
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```
