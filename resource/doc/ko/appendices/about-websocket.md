# 웹소켓 프로토콜

현재 Workerman의 웹 소켓 프로토콜 버전은 13입니다.

웹소켓 프로토콜은 HTML5의 새로운 프로토콜입니다. 이는 브라우저와 서버 간의 양방향 통신을 구현합니다.

## 웹소켓과 TCP의 관계

웹소켓과 HTTP와 마찬가지로 응용 계층 프로토콜이며, 둘 다 TCP를 기반으로 전송됩니다. 웹소켓 자체는 소켓과 큰 관련이 없으며, 동일하게 취급해서는 안됩니다.

## 웹소켓 프로토콜 핸드셰이크

웹소켓 프로토콜은 핸드셰이크 과정이 있으며, 핸드셰이크 시 브라우저와 서버는 HTTP 프로토콜로 통신하며, Workerman에서는 핸드셰이크 과정에 이렇게 개입할 수 있습니다.

**Workerman 버전이 4.1 일 때**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Connection\TcpConnection;
use Workerman\Worker;

$ws = new Worker('websocket://0.0.0.0:8181');
$ws->onConnect = function($connection)
{
    $connection->onWebSocketConnect = function($connection , $httpBuffer)
    {
        // 연결의 출처를 확인하고 정당하지 않으면 연결을 종료할 수 있습니다.
        // $_SERVER['HTTP_ORIGIN']는 웹소켓 연결을 시작한 페이지가 어디에서 왔는지를 나타냅니다.
        if($_SERVER['HTTP_ORIGIN'] != 'https://www.workerman.net')
        {
            $connection->close();
        }
        // onWebSocketConnect 내에서 $_GET $_SERVER를 사용할 수 있습니다.
        // var_dump($_GET, $_SERVER);
    };
};
Worker::runAll();
```

**Workerman 버전이 5.0 이상일 때**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Worker;

$worker = new Worker('websocket://0.0.0.0:12345');
$worker->onWebSocketConnect = function (TcpConnection $connection, Request $request) {
    if ($request->header('origin') != 'https://www.workerman.net') {
        $connection->close();
    }
    var_dump($request->get());
    var_dump($request->header());
};
Worker::runAll();
```

## 웹소켓 프로토콜로 이진 데이터 전송

웹소켓 프로토콜은 기본적으로 UTF-8 텍스트만 전송할 수 있지만, 이진 데이터를 전송하려면 다음 부분을 읽어보세요.

웹소켓 프로토콜에서는 프로토콜 헤더에 전송되는 데이터가 이진 데이터인지 UTF-8 텍스트 데이터인지를 나타내는 플래그를 사용합니다. 브라우저는 플래그와 전송된 콘텐츠 유형을 검증하고, 일치하지 않으면 오류를 표시하고 연결을 종료합니다.

따라서 서버에서 데이터를 전송할 때, 전송되는 데이터의 유형에 따라 이 플래그를 설정해야 합니다. 일반적인 UTF-8 텍스트인 경우에는 기본으로 설정되어 있으므로 추가적인 설정이 필요하지 않습니다.
```php
use Workerman\Protocols\Websocket;
$connection->websocketType = Websocket::BINARY_TYPE_BLOB;
```

이진 데이터인 경우에는 다음과 같이 설정해야 합니다.
```php
use Workerman\Protocols\Websocket;
$connection->websocketType = Websocket::BINARY_TYPE_ARRAYBUFFER;
```

**주의**: $connection->websocketType을 설정하지 않으면 기본적으로는 BINARY_TYPE_BLOB(즉, UTF-8 텍스트 유형)으로 설정됩니다. 일반적으로 응용 프로그램에서 전송되는 데이터는 UTF-8 텍스트입니다. 예를 들어, JSON 데이터를 전송하는 경우에는 추가 설정이 필요하지 않습니다. 이 속성은 이미지 데이터나 프로토버퍼 데이터와 같은 이진 데이터를 전송하는 경우에만 BINARY_TYPE_ARRAYBUFFER로 설정해야 합니다.

## Workerman을 웹소켓 클라이언트로 사용

[AsyncTcpConnection 클래스](../async-tcp-connection.md)와 [웹소켓 프로토콜(ws)](about-ws.md)을 사용하여 Workerman을 웹소켓 클라이언트로 구현하여 원격 웹소켓 서버에 연결하여 양방향 실시간 통신을 수행할 수 있습니다.
