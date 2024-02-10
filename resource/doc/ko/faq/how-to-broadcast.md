# 데이터를 브로드캐스트하는 방법

## 예시 (타이머를 이용한 브로드캐스팅)

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:2020');
// 이 예시에서는 프로세스 수를 1로 설정해야 합니다.
$worker->count = 1;
// 프로세스가 시작될 때 타이머를 설정하여 주기적으로 모든 클라이언트 연결에 데이터를 전송합니다.
$worker->onWorkerStart = function ($worker) {
    // 10초마다 타이머를 설정
    Timer::add(10, function () use ($worker) {
        // 현재 프로세스에 연결된 모든 클라이언트를 루프하여 현재 서버의 시간을 전송합니다.
        foreach ($worker->connections as $connection) {
            $connection->send(time());
        }
    });
};

// worker 실행
Worker::runAll();
```

## 예시 (그룹 채팅)

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:2020');
// 이 예시에서는 프로세스 수를 1로 설정해야 합니다.
$worker->count = 1;
// 클라이언트가 메시지를 보낼 때 다른 사용자에게 브로드캐스트합니다.
$worker->onMessage = function (TcpConnection $connection, $message) use ($worker) {
    foreach ($worker->connections as $connection) {
        $connection->send($message);
    }
};
// worker 실행
Worker::runAll();
```

## 설명:
**단일 프로세스:**
위의 예시들은 **단일 프로세스**(```$worker->count=1```)에서만 동작합니다. 여러 프로세스일 때, 여러 클라이언트가 서로 다른 프로세스에 연결될 수 있기 때문에, 프로세스 간 클라이언트는 격리되어 직접 통신할 수 없습니다. 즉, A 프로세스에서는 B 프로세스의 클라이언트 연결 객체에 직접적으로 데이터를 전송할 수 없습니다. (이를 위해서는 프로세스 간 통신이 필요하며, 예를 들어 Channel 구성요소를 사용할 수 있습니다. 예: [클러스터 전송 예시](../components/channel-examples.md), [그룹 전송 예시](../components/channel-examples2.md)).

**GatewayWorker를 사용하는 것을 권장함**
workerman을 기반으로 한 GatewayWorker 프레임워크는 그룹 채팅 및 브로드캐스팅과 같은 더 편리한 푸시 메커니즘을 제공하며, 다중 프로세스 및 다중 서버 배포가 가능합니다. 클라이언트에 데이터를 푸시해야 한다면 GatewayWorker 프레임워크를 사용하는 것이 좋습니다.

GatewayWorker 매뉴얼 주소: https://doc2.workerman.net
