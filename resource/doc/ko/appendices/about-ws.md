# 웹소켓 프로토콜

현재 Workerman의 **웹소켓 프로토콜 버전은 13**입니다.

웹소켓 클라이언트로서 Workerman은 ws 프로토콜을 통해 원격 웹소켓 서버에 연결하여 양방향 통신을 구현할 수 있습니다.

> **주의**
> ws 프로토콜은 AsyncTcpConnection을 통해 클라이언트로만 사용할 수 있으며, 웹소켓 서버에서는 사용할 수 없습니다. 다음과 같은 방식은 잘못된 방법입니다.

```php
$worker = new Worker('ws://0.0.0.0:8080');
```

Workerman을 웹소켓 서버로 사용하려면 [웹소켓 프로토콜](about-websocket.md)을 사용하십시오.

**ws 웹소켓 클라이언트 프로토콜 예시:**

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Protocols\Ws;
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;

$worker = new Worker();
// 프로세스 시작 시
$worker->onWorkerStart = function()
{
    // 원격 웹소켓 서버에 웹소켓 프로토콜로 연결
    $ws_connection = new AsyncTcpConnection("ws://127.0.0.1:1234");
    // 55초마다 서버에 opcode가 0x9인 웹소켓 하트비트를 보냄 (선택 사항)
    $ws_connection->websocketPingInterval = 55;
    // HTTP 헤더 설정 (선택 사항)
    $ws_connection->headers = [
        'Cookie' => 'PHPSID=82u98fjhakfusuanfnahfi; token=2hf9a929jhfihaf9i',
        'OtherKey' => 'values'
    ];
    // 데이터 유형 설정 (선택 사항)
    $ws_connection->websocketType = Ws::BINARY_TYPE_BLOB; // BINARY_TYPE_BLOB는 텍스트, BINARY_TYPE_ARRAYBUFFER는 이진
    // TCP가 3-way 핸드쉐이크를 완료한 후 (선택 사항)
    $ws_connection->onConnect = function($connection){
        echo "tcp connected\n";
    };
    // 웹소켓 핸드쉐이크를 완료한 후 (선택 사항)
    $ws_connection->onWebSocketConnect = function(AsyncTcpConnection $con, $response) {
        echo $response;
        $con->send('hello');
    };
    // 원격 웹소켓 서버로부터 메시지를 수신한 경우
    $ws_connection->onMessage = function($connection, $data){
        echo "recv: $data\n";
    };
    // 연결 오류가 발생한 경우, 일반적으로 원격 웹소켓 서버 연결 실패 오류임 (선택 사항)
    $ws_connection->onError = function($connection, $code, $msg){
        echo "error: $msg\n";
    };
    // 원격 웹소켓 서버 연결이 끊긴 경우 (선택 사항, 다시 연결하는 것을 권장함)
    $ws_connection->onClose = function($connection){
        echo "connection closed and try to reconnect\n";
        // 연결이 끊긴 경우 1초 후 다시 연결
        $connection->reConnect(1);
    };
    // 위의 각종 콜백을 설정한 후, 연결 작업을 실행
    $ws_connection->connect();
};
Worker::runAll();
```

더 많은 정보는 [ws/wss 클라이언트로서](../faq/as-wss-client.md)를 참고하세요.
