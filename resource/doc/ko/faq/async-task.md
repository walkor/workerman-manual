# 비동기 작업 구현하기

**질문:**

번거로운 비즈니스를 비동기적으로 처리하고, 주 비즈니스가 장시간 차단되는 것을 피하려면 어떻게 해야 합니까? 예를 들어 1000명의 사용자에게 이메일을 보내야 하는데 이 과정은 매우 느리고 몇 초 동안 차단될 수 있으며, 이 과정에서 주요 프로세스가 차단될 경우 후속 요청에 영향을 미칠 수 있습니다. 이러한 번거로운 작업을 다른 프로세스에게 비동기적으로 처리하려면 어떻게 해야 합니까?

**답변:**

지역 또는 다른 서버 심지어 서버 클러스터에 미리 번거로운 비즈니스를 처리할 몇 가지 작업 프로세스를 설정할 수 있습니다. 작업 프로세스 수를 CPU의 10배 정도 추가로 열 수 있으며, 그런 다음 호출자는 AsyncTcpConnection을 사용하여 데이터를 이러한 작업 프로세스에 비동기적으로 보내고 비동기적으로 처리 결과를 얻을 수 있습니다.

작업 프로세스 서버 측
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 작업 워커, Text 프로토콜 사용
$task_worker = new Worker('Text://0.0.0.0:12345');
// 필요에 따라 작업 프로세스 수를 추가로 열 수 있습니다.
$task_worker->count = 100;
$task_worker->name = 'TaskWorker';
$task_worker->onMessage = function(TcpConnection $connection, $task_data)
{
     // 가정: JSON 데이터가 전송됨
     $task_data = json_decode($task_data, true);
     //task_data에 따라 작업 로직을 처리하여 결과를 얻음... 여기서는 생략됨....
     $task_result = ......
     // 결과 전송
     $connection->send(json_encode($task_result));
};
Worker::runAll();
```

Workerman에서 호출하기
```php
use Workerman\Worker;
use \Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';


$worker = new Worker('websocket://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $ws_connection, $message)
{
    // 원격 작업 서비스와 비동기적 연결을 설정, IP는 원격 작업 서비스의 IP이며, 로컬이면 127.0.0.1, 클러스터라면 LVS의 IP입니다.
    $task_connection = new AsyncTcpConnection('Text://127.0.0.1:12345');
    // 작업 및 매개변수 데이터
    $task_data = array(
        'function' => 'send_mail',
        'args'       => array('from'=>'xxx', 'to'=>'xxx', 'contents'=>'xxx'),
    );
    // 데이터 전송
    $task_connection->send(json_encode($task_data));
    // 비동기적으로 결과 얻기
    $task_connection->onMessage = function(AsyncTcpConnection $task_connection, $task_result)use($ws_connection)
    {
         // 결과
         var_dump($task_result);
         // 결과를 받은 후 비동기 연결을 닫으십시오
         $task_connection->close();
         // 해당 웹소켓 클라이언트에 작업 완료 알림
         $ws_connection->send('작업 완료');
    };
    // 비동기 연결 실행
    $task_connection->connect();
};

Worker::runAll();
```

이렇게 하면 번거로운 작업을 로컬 또는 다른 서버의 프로세스에 맡기게 되며, 작업이 완료되면 비동기적으로 결과를 받을 수 있어서 비즈니스 프로세스가 차단되지 않습니다.
