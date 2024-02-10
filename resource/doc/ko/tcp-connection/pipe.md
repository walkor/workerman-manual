# 파이프
## 설명:
```php
void Connection::pipe(TcpConnection $target_connection)
```

## 매개변수
현재 연결의 데이터 스트림을 대상 연결로 전송합니다. 내부적으로 트래픽 제어가 내장되어 있어 TCP 프록시에 매우 유용합니다.

## 예제 TCP 프록시
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8483');
$worker->count = 12;

// TCP 연결 후
$worker->onConnect = function(TcpConnection $connection)
{
    // 로컬 80번 포트에 대한 비동기 연결 설정
    $connection_to_80 = new AsyncTcpConnection('tcp://127.0.0.1:80');
    // 현재 클라이언트 연결의 데이터를 80번 포트 연결로 보내도록 설정
    $connection->pipe($connection_to_80);
    // 80번 포트 연결이 반환하는 데이터를 클라이언트 연결로 보내도록 설정
    $connection_to_80->pipe($connection);
    // 비동기 연결 실행
    $connection_to_80->connect();
};

// worker 실행
Worker::runAll();
```
