# 식별자

## 설명:
```php
int Connection::$id
```

연결의 식별자입니다. 이는 자체 증가하는 정수입니다.

참고: workerman은 다중 프로세스로, 각 프로세스는 자체적으로 증가하는 연결 식별자를 유지하기 때문에 다중 프로세스 간에 연결 식별자가 중복될 수 있습니다. 중복되지 않는 연결 식별자를 원한다면 필요에 따라 connection->id에 worker->id 접두사를 추가하여 다시 할당할 수 있습니다.

## 참고
[Worker의 connections 속성](../worker/connections.md)

## 예시

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo $connection->id;
};
// worker 실행
Worker::runAll();
```
