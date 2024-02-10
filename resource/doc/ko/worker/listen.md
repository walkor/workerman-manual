# 듣기
```php
void Worker::listen(void)
```
Worker를 인스턴스화 한 후에 실행하여 듣게 함.

이 메소드는 Worker 프로세스가 시작된 후에 새로운 Worker 인스턴스를 동적으로 생성하는 데 사용되며, 동일한 프로세스에서 여러 포트를 듣고 여러 프로토콜을 지원할 수 있음. 이 방법을 사용하면 현재 프로세스에서만 듣기를 추가하며, 새로운 프로세스를 동적으로 생성하거나 onWorker.Start 메소드를 실행하지 않음에 주의해야 함.

예를 들어, HTTP Worker를 시작한 후 WebSocket Worker를 인스턴스화하면 해당 프로세스는 HTTP 프로토콜로 액세스할 수 있고 WebSocket 프로토콜로 액세스할 수 있음. WebSocket Worker와 HTTP Worker가 같은 프로세스에 있으므로 둘은 공통의 메모리 변수에 액세스하고 모든 소켓 연결을 공유할 수 있음. 이는 HTTP 요청을 수신한 후 WebSocket 클라이언트를 조작하여 클라이언트에 데이터를 푸시하는 것과 같은 효과를 얻을 수 있음.

**주의:**

PHP 버전이 <=7.0인 경우에는 동일한 포트의 Worker를 여러 자식 프로세스에서 인스턴스화할 수 없음. 예를 들어, A 프로세스가 2016 포트를 듣는 Worker를 만들었다면, B 프로세스에서는 2016 포트를 듣는 Worker를 더 이상 만들 수 없으며, 그렇지 않을 경우 ```Address already in use``` 오류가 발생함. 아래의 코드는 ```동작하지 않음```.

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
// 4 개의 프로세스
$worker->count = 4;
// 각 프로세스가 시작되면 현재 프로세스에 새로운 Worker를 듣게 함
$worker->onWorkerStart = function($worker)
{
    /**
     * 4 개의 프로세스가 시작될 때 2016 포트의 Worker를 만듬
     * worker->listen()을 실행하면 Address already in use 오류가 발생함
     * 만약 worker->count=1로 설정하면 오류가 발생하지 않음
     */
    $inner_worker = new Worker('http://0.0.0.0:2016');
    $inner_worker->onMessage = 'on_message';
    // 리스닝 실행. 여기서는 Address already in use 오류가 발생함
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

// 모든 worker 실행
Worker::runAll();
```

PHP 버전이 >=7.0일 경우에는 Worker->reusePort=true로 설정하여 여러 자식 프로세스가 동일한 포트의 Worker를 만들 수 있음. 아래의 예시 참조:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2015');
// 4 개의 프로세스
$worker->count = 4;
// 각 프로세스가 시작되면 현재 프로세스에 새로운 Worker를 듣게 함
$worker->onWorkerStart = function($worker)
{
    $inner_worker = new Worker('http://0.0.0.0:2016');
    // 포트 재사용 설정, 동일한 포트를 듣는 Worker를 만들 수 있음 (PHP>=7.0 필요)
    $inner_worker->reusePort = true;
    $inner_worker->onMessage = 'on_message';
    // 리스닝 실행. 정상 리스닝, 오류가 발생하지 않음
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

// 모든 worker 실행
Worker::runAll();
```
### 예시: PHP 백엔드에서 클라이언트에 실시간으로 메시지 푸시하기

**원리:**

1. WebSocket Worker를 설정하여 클라이언트와의 장기간 연결을 유지함
2. WebSocket Worker 내부에 텍스트 Worker를 생성함
3. WebSocket Worker와 텍스트 Worker는 같은 프로세스이므로 클라이언트 연결을 쉽게 공유할 수 있음
4. 특정 PHP 백그라운드 시스템은 텍스트 프로토콜을 사용하여 텍스트 Worker와 통신함
5. 텍스트 Worker가 WebSocket 연결을 조작하여 데이터를 푸시함

**코드 및 단계:**

push.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 1234 포트를 듣는 Worker 컨테이너 초기화
$worker = new Worker('websocket://0.0.0.0:1234');

/*
 * 여기서 프로세스 수는 1로 설정되어야 함
 */
$worker->count = 1;
// worker 프로세스가 시작된 후 내부 통신 포트를 열기 위해 text Worker를 만듬
$worker->onWorkerStart = function($worker)
{
    // 내부 포트를 연다. 내부 시스템이 데이터를 푸시하기 쉽도록 함. Text 프로토콜 형식은 텍스트+줄바꿈 문자임
    $inner_text_worker = new Worker('text://0.0.0.0:5678');
    $inner_text_worker->onMessage = function(TcpConnection $connection, $buffer)
    {
        // $data 배열 형식, uid가 있는 경우 해당 uid의 페이지로 데이터를 푸시함
        $data = json_decode($buffer, true);
        $uid = $data['uid'];
        // Workerman을 통해 uid의 페이지로 데이터를 푸시함
        $ret = sendMessageByUid($uid, $buffer);
        // 푸시 결과 반환
        $connection->send($ret ? 'ok' : 'fail');
    };
    // ## 리스닝 실행 ##
    $inner_text_worker->listen();
};
//uidConnections 배열 추가하여 uid를 연결에 저장함
$worker->uidConnections = array();
// 클라이언트가 메시지를 보낼 때 실행할 콜백 함수
$worker->onMessage = function(TcpConnection $connection, $data)
{
    global $worker;
    // 현재 클라이언트가 이미 인증되었는지, 즉 uid가 설정되었는지 확인함
    if(!isset($connection->uid))
    {
       // 인증되지 않은 경우 첫 번째 패킷을 uid로 처리함 (여기서는 편의상 진정한 인증은 수행하지 않음)
       $connection->uid = $data;
       /* uid를 연결에 저장함. 이렇게하면 uid로 연결을 쉽게 찾아 연결에 특정 uid에 대한 데이터를 푸시할 수 있음
        */
       $worker->uidConnections[$connection->uid] = $connection;
       return;
    }
};

// 클라이언트 연결이 종료될 때
$worker->onClose = function(TcpConnection $connection)
{
    global $worker;
    if(isset($connection->uid))
    {
        // 연결이 종료될 때 맵을 삭제함
        unset($worker->uidConnections[$connection->uid]);
    }
};

// 모든 인증된 사용자에게 데이터 푸시
function broadcast($message)
{
   global $worker;
   foreach($worker->uidConnections as $connection)
   {
        $connection->send($message);
   }
}

// uid에 대한 데이터 푸시
function sendMessageByUid($uid, $message)
{
    global $worker;
    if(isset($worker->uidConnections[$uid]))
    {
        $connection = $worker->uidConnections[$uid];
        $connection->send($message);
        return true;
    }
    return false;
}

// 모든 worker 실행
Worker::runAll();
```

백엔드 서비스 시작
```sh
php push.php start -d
```

클라이언트에서 메시지 수신하는 JS 코드
```javascript
var ws = new WebSocket('ws://127.0.0.1:1234');
ws.onopen = function(){
    var uid = 'uid1';
    ws.send(uid);
};
ws.onmessage = function(e){
    alert(e.data);
};
```

백엔드에서 메시지 푸시하는 코드
```php
// 내부 푸시 포트로 소켓 연결 설정
$client = stream_socket_client('tcp://127.0.0.1:5678', $errno, $errmsg, 1);
// 포함된 uid 필드를 기준으로 푸시하는 데이터
$data = array('uid'=>'uid1', 'percent'=>'88%');
// 데이터 전송. 5678 포트는 Text 프로토콜 포트이므로 데이터 끝에 줄바꿈 문자를 추가해야 함
fwrite($client, json_encode($data)."\n");
// 푸시 결과 읽기
echo fread($client, 8192);
```
