# 심장박동

참고: 장기 연결 어플리케이션에는 심장박동이 필요합니다. 그렇지 않으면 일부 노드에서 긴 시간 동안 통신이 없는 연결이 강제로 끊길 수 있습니다.

심장박동의 주요 목적은 다음과 같습니다.

1. 클라이언트는 정기적으로 서버에 데이터를 보내어, 통신이 오랫동안 없어지는 것을 방지하여 방화벽 등의 일부 노드에 의해 연결이 끊어지는 상황을 방지합니다.

2. 서버는 심장박동을 통해 클라이언트의 온라인 상태를 판단할 수 있으며, 클라이언트가 규정된 시간 내에 데이터를 보내지 않으면 클라이언트를 오프라인으로 간주합니다. 이를 통해 극단적인 상황(정전, 네트워크 이상 등)으로 인해 클라이언트가 오프라인임을 감지할 수 있습니다.

심장박동 간격 권장 값:

클라이언트가 심장박동을 보내는 간격은 60초보다 짧게하는 것이 좋습니다. 예를 들어 55초로 설정할 수 있습니다.

> 심장박동의 데이터 형식에 대한 특별한 요구사항은 없으며, 서버가 인식할 수 있는 형식이면 됩니다.

## 심장박동 예시
```php
<?php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 심장박동 간격 55초
define('HEARTBEAT_TIME', 55);

$worker = new Worker('text://0.0.0.0:1234');

$worker->onMessage = function(TcpConnection $connection, $msg) {
    // connection에 마지막 메시지를 받은 시간을 기록하는 lastMessageTime 속성을 임시로 설정합니다.
    $connection->lastMessageTime = time();
    // 다른 비즈니스 로직...
};

// 프로세스가 시작된 후 10초마다 실행되는 타이머 설정
$worker->onWorkerStart = function($worker) {
    Timer::add(10, function()use($worker){
        $time_now = time();
        foreach($worker->connections as $connection) {
            // 이 connection이 아직 메시지를 받지 않았을 수도 있으므로, lastMessageTime을 현재 시간으로 설정합니다.
            if (empty($connection->lastMessageTime)) {
                $connection->lastMessageTime = $time_now;
                continue;
            }
            // 마지막 통신 시간 간격이 심장박동 간격을 초과하면 클라이언트가 이미 오프라인으로 간주되어 연결을 닫습니다.
            if ($time_now - $connection->lastMessageTime > HEARTBEAT_TIME) {
                $connection->close();
            }
        }
    });
};

Worker::runAll();
```

위의 설정은 클라이언트가 55초 이상 데이터를 서버에 보내지 않으면 서버가 클라이언트를 오프라인으로 간주하여 연결을 닫고 onClose를 트리거하도록합니다.

## 연결 끊김 재연결(중요)

클라이언트가 심장박동을 보내거나 서버가 심장박동을 보내더라도 연결은 끊길 수 있습니다. 예를 들어, 브라우저가 최소화되어 JavaScript가 일시 중지되었거나, 브라우저가 다른 탭 페이지로 전환되어 JavaScript가 일시 중지되었거나, 컴퓨터가 절전 모드로 들어간 경우 등이 있습니다. 이외에도 모바일에서 네트워크 변경, 신호 약화, 핸드폰 화면 꺼짐, 핸드폰 애플리케이션 백그라운드 전환, 라우터 장애, 비즈니스가 연결을 종료하는 등의 사례들이 있습니다. 특히 외부 네트워크 환경에서는 많은 노드가 1분간 활동이 없는 연결을 정리하는데, 이것이 심장박동 간격을 1분 미만으로 권장하는 이유입니다.

외부 네트워크 환경에서 연결이 쉽게 끊길 수 있으므로 연결 재연결은 장기 연결 앱이 갖추어야 하는 기능입니다(연결 재연결은 클라이언트만 할 수 있으며, 서버는 구현할 수 없습니다). 예를 들어, 브라우저의 웹소켓은 onclose 이벤트를 청취하여 onclose가 발생하면 새 연결을 설정해야 합니다(중단되는 것을 피하기 위해 연결 설정을 지연시킬 수 있습니다). 더 엄격하게는 서버도 주기적으로 심장박동 데이터를 보내야 하며, 클라이언트는 정기적으로 서버의 심장박동 데이터가 시간 초과되었는지 확인해야하며, 규정된 시간 내에 서버의 심장박동 데이터를받지 않았다면 연결이 이미 끊어졌으며, 닫혀있는 연결을 다시 설정해야합니다.
