# 예시 1
**``` (Workerman 버전>=3.3.0이 필요합니다) ```**

Worker를 기반으로 한 다중 프로세스(분산 클러스터) 푸시 시스템, 클러스터 그룹 메시지 보내기, 클러스터 브로드캐스트.

`start_channel.php`
시스템 전체에는 하나의 start_channel 서비스만 배포 할 수 있습니다. 192.168.1.1에서 실행되었다고 가정합니다.
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// 채널 서버 초기화
$channel_server = new Channel\Server('0.0.0.0', 2206);

Worker::runAll();
```

`start_ws.php`
시스템 전체에는 여러 개의 start_ws 서비스를 배포할 수 있으며, 각각 192.168.1.2 및 192.168.1.3 서버에서 실행되었다고 가정합니다.
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 웹소켓 서버
$worker = new Worker('websocket://0.0.0.0:4236');
$worker->count=2;
$worker->name = 'pusher';
$worker->onWorkerStart = function($worker)
{
    // 채널 클라이언트가 채널 서버에 연결합니다.
    Channel\Client::connect('192.168.1.1', 2206);
    // 자신의 프로세스 id를 이벤트 이름으로 사용
    $event_name = $worker->id;
    // worker->id 이벤트를 구독하고 이벤트 처리 함수를 등록합니다.
    Channel\Client::on($event_name, function($event_data)use($worker){
        $to_connection_id = $event_data['to_connection_id'];
        $message = $event_data['content'];
        if(!isset($worker->connections[$to_connection_id]))
        {
            echo "연결이 존재하지 않습니다\n";
            return;
        }
        $to_connection = $worker->connections[$to_connection_id];
        $to_connection->send($message);
    });

    // 브로드캐스트 이벤트를 구독합니다.
    $event_name = '브로드캐스트';
    // 브로드캐스트 이벤트를 수신한 경우 현재 프로세스 내의 모든 클라이언트 연결에 브로드캐스트 데이터를 전송합니다.
    Channel\Client::on($event_name, function($event_data)use($worker){
        $message = $event_data['content'];
        foreach($worker->connections as $connection)
        {
            $connection->send($message);
        }
    });
};

$worker->onConnect = function(TcpConnection $connection)use($worker)
{
    $msg = "workerID:{$worker->id} connectionID:{$connection->id} connected\n";
    echo $msg;
    $connection->send($msg);
};
Worker::runAll();
```


`start_http.php`
시스템 전체에는 여러 개의 start_ws 서비스를 배포할 수 있으며, 각각 192.168.1.4 및 192.168.1.5 서버에서 실행되었다고 가정합니다.
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// http 요청을 처리하고 임의의 클라이언트에 데이터를 푸시합니다. workerID 및 connectionID를 전달해야 합니다.
$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->name = 'publisher';
$http_worker->onWorkerStart = function()
{
    Channel\Client::connect('192.168.1.1', 2206);
};
$http_worker->onMessage = function(TcpConnection $connection, $request)
{
    // workerman 4.x와 호환되게합니다.
    if (!is_array($request)) {
            $_GET = $request->get();
   }
    $connection->send('ok');
    if(empty($_GET['content'])) return;
    // 특정 worker 프로세스의 특정 연결에 데이터를 푸시합니다.
    if(isset($_GET['to_worker_id']) && isset($_GET['to_connection_id']))
    {
        $event_name = $_GET['to_worker_id'];
        $to_connection_id = $_GET['to_connection_id'];
        $content = $_GET['content'];
        Channel\Client::publish($event_name, array(
           'to_connection_id' => $to_connection_id,
           'content'          => $content
        ));
    }
    // 글로벌 브로드캐스트 데이터입니다.
    else
    {
        $event_name = '브로드캐스트';
        $content = $_GET['content'];
        Channel\Client::publish($event_name, array(
           'content'          => $content
        ));
    }
};

Worker::runAll();

```

## 테스트
1. 각 서버에서 서비스 실행
2. 클라이언트가 서버에 연결합니다.

크롬 브라우저를 열고 F12를 눌러 디버그 콘솔을 열고, Console 탭에 입력하십시오 (또는 아래 코드를 HTML 페이지에 넣어 JavaScript로 실행합니다).

```javascript
// ws://192.168.1.3:4236에도 연결할 수 있습니다.
ws = new WebSocket("ws://192.168.1.2:4236");
ws.onmessage = function(e) {
    alert("서버로부터 메시지를 수신했습니다:" + e.data);
};
```

3. HTTP 인터페이스를 통해 푸시합니다.

url을 방문하여 다음 작업을 수행합니다. ```http://192.168.1.4:4237/?content={$content}```  또는  ```http://192.168.1.5:4237/?content={$content}```를 사용하여 모든 클라이언트 연결에 ```$content``` 데이터를 푸시합니다.

url을 방문하여 다음 작업을 수행합니다. ```http://192.168.1.4:4237/?to_worker_id={$worker_id}&to_connection_id={$connection_id}&content={$content}``` 또는 ```http://192.168.1.5:4237/?to_worker_id={$worker_id}&to_connection_id={$connection_id}&content={$content}```를 사용하여 특정 worker 프로세스의 특정 클라이언트 연결에 ```$content``` 데이터를 푸시합니다.

참고: 테스트 중에 ```{$worker_id}```, ```{$connection_id}``` 및 ```{$content}```를 실제 값으로 대체하세요.
