``` (Workerman 버전>=3.3.0이 필요함) ```
```php
void \Channel\Client::on(string $event_name, callback $callback_function)
```

```$event_name``` 이벤트를 구독하고 발생 시 ```$callback_function``` 콜백을 등록합니다.

## 콜백 함수 매개변수

``` $event_name ```

구독된 이벤트의 이름으로, 임의의 문자열일 수 있습니다.

 ``` $callback_function ```

이벤트가 발생할 때 트리거되는 콜백 함수입니다. 함수 원형은 ```callback_function(mixed $event_data)```입니다. ```$event_data```는 이벤트가 발생할 때 전달된 이벤트 데이터입니다.


참고:

동일한 이벤트에 두 개의 콜백 함수가 등록된 경우, 후속 콜백 함수가 이전 콜백 함수를 덮어씁니다.

## 예제
멀티 프로세스 Worker (멀티 서버), 하나의 클라이언트가 메시지를 보내면 모든 클라이언트에게 브로드캐스트합니다.


start.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 채널 서버 초기화
$channel_server = new Channel\Server('0.0.0.0', 2206);

// 웹소켓 서버
$worker = new Worker('websocket://0.0.0.0:4236');
$worker->name = 'websocket';
$worker->count = 6;
// 각 worker 프로세스 시작 시
$worker->onWorkerStart = function($worker)
{
    // 채널 클라이언트가 채널 서버에 연결
    Channel\Client::connect('127.0.0.1', 2206);
    // broadcast 이벤트를 구독하고 이벤트 콜백 등록
    Channel\Client::on('broadcast', function($event_data)use($worker){
        // 현재 worker 프로세스의 모든 클라이언트에게 메시지를 브로드캐스트합니다.
        foreach($worker->connections as $connection)
        {
            $connection->send($event_data);
        }
    });
};

$worker->onMessage = function(TcpConnection $connection, $data)
{
   // 클라이언트가 보낸 데이터를 이벤트 데이터로 취급
   $event_data = $data;
   // 모든 worker 프로세스에게 broadcast 이벤트를 발행합니다.
   \Channel\Client::publish('broadcast', $event_data);
};

Worker::runAll();
```

**테스트**

Chrome 브라우저를 열고, F12를 눌러 개발자 도구를 열고, Console 탭에 다음을 입력하세요 (또는 아래 코드를 html 페이지에 넣고 JavaScript로 실행하세요)

메시지를 수신하는 연결
```javascript
// 127.0.0.1을 실제 Workerman이 있는 IP로 바꿔주세요.
ws = new WebSocket("ws://127.0.0.1:4236");
ws.onmessage = function(e) {
    alert("서버로부터의 메시지를 받았습니다: " + e.data);
};
```

메시지를 브로드캐스트합니다.
```
ws.send('안녕하세요');
```
