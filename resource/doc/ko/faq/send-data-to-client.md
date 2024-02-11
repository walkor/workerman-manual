Workerman에서 특정 클라이언트에게 데이터를 전송하는 방법
GatewayWorker를 사용하지 않고 worker를 사용하여 서버를 구성하고, 특정 사용자에게 메시지를 푸시하는 방법을 실행할 수 있습니까?

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 1234번 포트를 수신하는 worker 컨테이너를 초기화합니다.
$worker = new Worker('websocket://workerman.net:1234');
// ====여기서 프로세스 수는 꼭 1로 설정해야 합니다====
$worker->count = 1;
// uid를 connection으로 매핑하는 속성을 추가합니다(uid는 사용자 ID 또는 클라이언트의 고유 식별자입니다)
$worker->uidConnections = array();
// 클라이언트가 메시지를 보낼 때 실행되는 콜백 함수
$worker->onMessage = function(TcpConnection $connection, $data)
{
    global $worker;
    // 현재 클라이언트가 인증되었는지 확인합니다. 즉, uid가 설정되어 있는지 확인합니다.
    if(!isset($connection->uid))
    {
       // 인증되지 않았다면, 첫 번째 패킷을 uid로 사용합니다(여기서는 실제 인증을 쉽게하기 위해 사용되지 않습니다).
       $connection->uid = $data;
       /* uid를 connection으로 매핑하여 uid를 통해 connection을 쉽게 찾을 수 있게 합니다.
        * 특정 uid에 대한 데이터를 푸시하는 기능을 구현합니다.
        */
       $worker->uidConnections[$connection->uid] = $connection;
       return $connection->send('login success, your uid is ' . $connection->uid);
    }
    // 다른 로직을 사용하여 특정 uid에게 메시지를 보내거나 전체 브로드캐스트를 수행합니다.
    // 메시지 형식: uid:message에 따라 uid에 message를 보냅니다.
    // uid가 all인 경우 전체 브로드캐스트를 수행합니다.
    list($recv_uid, $message) = explode(':', $data);
    // 전체 브로드캐스트
    if($recv_uid == 'all')
    {
        broadcast($message);
    }
    // 특정 uid에게 전송
    else
    {
        sendMessageByUid($recv_uid, $message);
    }
};

// 클라이언트 연결이 끊길 때
$worker->onClose = function(TcpConnection $connection)
{
    global $worker;
    if(isset($connection->uid))
    {
        // 연결이 끊기면 매핑을 삭제합니다.
        unset($worker->uidConnections[$connection->uid]);
    }
};

// 인증된 모든 사용자에게 데이터를 푸시합니다.
function broadcast($message)
{
   global $worker;
   foreach($worker->uidConnections as $connection)
   {
        $connection->send($message);
   }
}

// uid에 대한 데이터를 푸시합니다.
function sendMessageByUid($uid, $message)
{
    global $worker;
    if(isset($worker->uidConnections[$uid]))
    {
        $connection = $worker->uidConnections[$uid];
        $connection->send($message);
    }
}

// 모든 worker를 실행합니다(실제로 하나의 worker만 정의되어 있습니다).
Worker::runAll();
```	

**참고:**

위 예제는 uid에 대한 푸시를 수행할 수 있으며, 단일 프로세스이지만 10만 개의 온라인 사용자를 지원하는 데 문제가 없습니다.


이 예제는 단일 프로세스만 지원되며($worker->count는 1이어야 합니다), 다중 프로세스 또는 서버 클러스터를 지원하려면 프로세스 간 통신을 완료하는 Channel 구성요소가 필요합니다. 개발도 매우 간단하며 [Channel 구성요소를 사용한 클러스터 푸시 예제](../components/channel-examples.md)를 참고할 수 있습니다.

**다른 시스템에서 클라이언트에게 메시지를 푸시하려면 [다른 프로젝트에서 푸시](push-in-other-project.md) 섹션을 참고하십시오**
