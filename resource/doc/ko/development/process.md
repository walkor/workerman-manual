# 기본 흐름
(간단한 웹소켓 채팅방 서버를 예로 들어)

#### 1. 프로젝트 디렉토리를 아무 곳에서 생성
예: SimpleChat/
디렉토리로 이동하여 `composer require workerman/workerman` 명령 실행

#### 2. `vendor/autoload.php`를 포함 (composer 설치 후 생성됨)
start.php 파일을 생성하고, `vendor/autoload.php`를 포함
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';
```

#### 3. 프로토콜 선택
여기서는 Text 텍스트 프로토콜을 선택합니다 (WorkerMan에서 자체 정의한 텍스트 + 개행 문자 형식의 프로토콜)

(현재 WorkerMan은 HTTP, 웹소켓, Text 텍스트 프로토콜을 지원하며, 다른 프로토콜을 사용해야 하는 경우, 프로토콜 장에서 직접 프로토콜을 개발해야 합니다.)

#### 4. 필요에 따라 진입 시작 스크립트 작성
아래는 간단한 채팅방의 진입 파일입니다.

SimpleChat/start.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$global_uid = 0;

// 클라이언트가 접속하면 uid를 할당하고 연결을 저장하고 모든 클라이언트에게 알립니다.
function handle_connection($connection)
{
    global $text_worker, $global_uid;
    // 이 연결에 uid 할당
    $connection->uid = ++$global_uid;
}

// 클라이언트가 메시지를 보내면 모두에게 전달
function handle_message(TcpConnection $connection, $data)
{
    global $text_worker;
    foreach($text_worker->connections as $conn)
    {
        $conn->send("user[{$connection->uid}] said: $data");
    }
}

// 클라이언트가 연결을 끊으면 모든 클라이언트에게 방송
function handle_close($connection)
{
    global $text_worker;
    foreach($text_worker->connections as $conn)
    {
        $conn->send("user[{$connection->uid}] logout");
    }
}

// 2347 포트에서 텍스트 프로토콜 Worker를 만듭니다.
$text_worker = new Worker("text://0.0.0.0:2347");

// 클라이언트 간 데이터 전송이 용이하도록 프로세스를 한 개만 시작합니다.
$text_worker->count = 1;

$text_worker->onConnect = 'handle_connection';
$text_worker->onMessage = 'handle_message';
$text_worker->onClose = 'handle_close';

Worker::runAll();

```


#### 5. 테스트
Text 프로토콜을 사용하여 telnet 명령으로 테스트할 수 있습니다.
```shell
telnet 127.0.0.1 2347
```
