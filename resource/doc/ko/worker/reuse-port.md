# reusePort
> **주의**
> workerman 버전>= 3.2.1  PHP>=7.0 이상이 필요하며, Windows 및 Mac OS 시스템은 이 기능을 지원하지 않습니다.

## 설명:

```php
bool Worker::$reusePort
```

현재 worker가 포트 재사용(socket의 SO_REUSEPORT 옵션)을 활성화할지 여부를 설정합니다.

포트 재사용을 활성화하면 관련이 없는 여러 프로세스가 동일한 포트를 수신할 수 있으며, 시스템 커널이 로드 밸런싱을 수행하여 소켓 연결을 어떤 프로세스가 처리할지 결정함으로써 공포 효과를 피하고 다중 프로세스 단일 연결 응용 프로그램의 성능을 향상시킬 수 있습니다.

**주의:** 이 기능은 PHP 버전>=7.0 이상이 필요합니다.

## 예제 1

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->count = 4;
$worker->reusePort = true;
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('ok');
};
// worker 실행
Worker::runAll();
```

## 예제 2: workerman 다중 포트(다중 프로토콜) 수신
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2015');
$worker->count = 4;
// 각 프로세스가 시작된 후 현재 프로세스에서 새로운 리스닝 추가
$worker->onWorkerStart = function($worker)
{
    $inner_worker = new Worker('http://0.0.0.0:2016');
    /**
     * 다중 프로세스가 동일한 포트를 수신하는 경우(부모 프로세스에서 상속되는 리스닝 소켓이 아님)
     * 포트 재사용을 활성화해야하며, 그렇지 않으면 Address already in use 오류가 발생할 수 있습니다
     */
    $inner_worker->reusePort = true;
    $inner_worker->onMessage = 'on_message';
    // 수신 실행
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

// worker 실행
Worker::runAll();
```
