# 프로토콜
요구 사항 ```(workerman >= 3.2.7)```

## 설명:
```php
string Worker::$protocol
```

현재 Worker 인스턴스의 프로토콜 클래스를 설정합니다.

참고: 프로토콜 처리 클래스는 Worker 초기화 및 리스닝 매개변수에서 직접 지정할 수 있습니다. 예를 들어
```php
$worker = new Worker('http://0.0.0.0:8686');
```

## 예시

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8686');
$worker->protocol = 'Workerman\\Protocols\\Http';

$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    $connection->send("hello");
};

// worker 실행
Worker::runAll();
```

위의 코드는 아래 코드와 동일합니다.

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

/**
 * 먼저 사용자 정의 \Protocols\Http 프로토콜 클래스를 확인하고,
 * 없는 경우 workerman 내장 프로토콜 클래스 Workerman\Protocols\Http를 사용합니다.
 */
$worker = new Worker('http://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    $connection->send("hello");
};

// worker 실행
Worker::runAll();
```
