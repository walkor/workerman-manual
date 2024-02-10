# 프로토콜

## 설명:
```php
string Connection::$protocol
```

현재 연결의 프로토콜 클래스를 설정합니다.

## 예시

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;

require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    $connection->protocol = 'Workerman\\Protocols\\Http';
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    // send를 호출하면 $connection->protocol::encode()가 자동으로 호출되어 데이터를 패킹한 후 전송됩니다.
    $connection->send("hello");
};
// worker 실행
Worker::runAll();
```
