# 간단한 개발 예제

## 설치

**workerman 설치**
빈 디렉토리에서 다음 명령어를 실행합니다.
`composer require workerman/workerman`

## 예제 1. HTTP 프로토콜을 사용하여 웹 서비스 제공

**start.php 파일 생성**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

// http 프로토콜 통신을 위해 2345 포트를 감시하는 Worker 생성
$http_worker = new Worker("http://0.0.0.0:2345");

// 4개의 프로세스를 시작하여 서비스를 제공합니다.
$http_worker->count = 4;

// 브라우저에서 데이터를 받았을 때 브라우저에 "hello world"를 응답합니다.
$http_worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // 브라우저에 "hello world"를 보냅니다.
    $connection->send('hello world');
};

// Worker 실행
Worker::runAll();
```

**명령줄 실행 (Windows 사용자는 [cmd 명령줄](https://baike.baidu.com/item/%E5%91%BD%E4%BB%A4%E6%8F%90%E7%A4%BA%E7%AC%A6?fromtitle=CMD&fromid=1193011&type=syn))**
```shell
php start.php start
```

**테스트**

서버 IP가 127.0.0.1인 경우

브라우저에서 http://127.0.0.1:2345 주소에 접속합니다.

**참고 사항:**

1. 접속에 문제가 발생하는 경우 [클라이언트 접속 실패 원인](../faq/client-connect-fail.md)을 참조하여 문제를 해결하십시오.
2. 서버는 http 프로토콜을 사용하므로 websocket 등 다른 프로토콜로는 직접 통신할 수 없습니다.

## 예제 2. WebSocket 프로토콜을 사용하여 서비스 제공

**ws_test.php 파일 생성**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 참고: 이 예제는 이전 예제와 다르게 websocket 프로토콜을 사용합니다.
$ws_worker = new Worker("websocket://0.0.0.0:2000");

// 4개의 프로세스를 시작하여 서비스를 제공합니다.
$ws_worker->count = 4;

// 클라이언트로부터 데이터를 받은 후 "hello $data"를 클라이언트에 반환합니다.
$ws_worker->onMessage = function(TcpConnection $connection, $data)
{
    // 클라이언트에게 "hello $data"를 보냅니다.
    $connection->send('hello ' . $data);
};

// Worker 실행
Worker::runAll();
```

**명령줄 실행**
```shell
php ws_test.php start
```

**테스트**

Chrome 브라우저를 열고 F12를 눌러 디버그 콘솔을 열고 Console 탭에서 다음을 입력합니다(js를 html 페이지에 넣어 실행할 수도 있음)

```javascript
// 서버 IP가 127.0.0.1인 경우
ws = new WebSocket("ws://127.0.0.1:2000");
ws.onopen = function() {
    alert("연결 성공");
    ws.send('tom');
    alert("서버에 문자열 'tom'을 보냅니다.");
};
ws.onmessage = function(e) {
    alert("서버에서의 메시지 수신: " + e.data);
};
```

**참고 사항:**

1. 접속에 문제가 발생하는 경우 [매뉴얼 FAQ-연결 실패](../faq/client-connect-fail.md)를 참조하여 문제를 해결하십시오.
2. 서버는 websocket 프로토콜을 사용하므로 http 등 다른 프로토콜로는 직접 통신할 수 없습니다.

## 예제 3. 직접 TCP를 사용하여 데이터 전송

**tcp_test.php 파일 생성**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 어플리케이션 계층 프로토콜을 사용하지 않고 2347 포트를 감시하는 Worker 생성
$tcp_worker = new Worker("tcp://0.0.0.0:2347");

// 4개의 프로세스를 시작하여 서비스를 제공합니다.
$tcp_worker->count = 4;

// 클라이언트에서 데이터를 받은 경우 "hello $data"를 클라이언트에 반환합니다.
$tcp_worker->onMessage = function(TcpConnection $connection, $data)
{
    // 클라이언트에게 "hello $data"를 보냅니다.
    $connection->send('hello ' . $data);
};

// Worker 실행
Worker::runAll();
```

**명령줄 실행**

```shell
php tcp_test.php start
```

**테스트: 명령줄 실행**
(아래는 리눅스 명령줄이며, Windows에서는 다를 수 있음)
```shell
telnet 127.0.0.1 2347
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
tom
hello tom
```

**참고 사항:**

1. 접속에 문제가 발생하는 경우 [매뉴얼 FAQ-연결 실패](../faq/client-connect-fail.md)를 참조하여 문제를 해결하십시오.
2. 서버는 순수한 tcp 프로토콜을 사용하므로 websocket, http 등 다른 프로토콜로는 직접 통신할 수 없습니다.
