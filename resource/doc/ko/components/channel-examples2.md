```php
# 예시 1
**``` (Workerman 버전 >=3.3.0 이상 요구) ```**

Worker를 기반으로 한 다중 프로세스 그룹 푸시 시스템

``` php
    use Workerman\Worker;
    use Workerman\Connection\TcpConnection;
    require_once __DIR__ . '/vendor/autoload.php';

    $channel_server = new Channel\Server('0.0.0.0', 2206);

    $worker = new Worker('websocket://0.0.0.0:1234');
    $worker->count = 8;
    // 전역 그룹 및 연결 매핑 배열
    $group_con_map = array();
    $worker->onWorkerStart = function(){
        // Channel 클라이언트가 Channel 서버에 연결
        Channel\Client::connect('127.0.0.1', 2206);

        // 전역 그룹 메시지 이벤트를 듣기
        Channel\Client::on('send_to_group', function($event_data){
            $group_id = $event_data['group_id'];
            $message = $event_data['message'];
            global $group_con_map;
            var_dump(array_keys($group_con_map));
            if (isset($group_con_map[$group_id])) {
                foreach ($group_con_map[$group_id] as $con) {
                    $con->send($message);
                }
            }
        });
    };
    $worker->onMessage = function(TcpConnection $con, $data){
        // 그룹 가입 메시지 {"cmd":"add_group", "group_id":"123"}  
        // 또는 그룹 메시지 전송 {"cmd":"send_to_group", "group_id":"123", "message":"이것은 메시지입니다"}
        $data = json_decode($data, true);
        var_dump($data);
        $cmd = $data['cmd'];
        $group_id = $data['group_id'];
        switch($cmd) {
            // 연결이 그룹에 가입
            case "add_group":
                global $group_con_map;
                // 연결을 해당 그룹 배열에 추가
                $group_con_map[$group_id][$con->id] = $con;
                // 이 연결이 어떤 그룹에 가입했는지 기록하여 onclose 시에 group_con_map에서 해당 그룹의 데이터를 정리할 수 있도록 함
                $con->group_id = isset($con->group_id) ? $con->group_id : array();
                $con->group_id[$group_id] = $group_id;
                break;
            // 그룹에 메시지 전송
            case "send_to_group":
                // Channel\Client를 통해 모든 서버의 모든 프로세스에 대해 그룹 메시지 이벤트를 발행
                Channel\Client::publish('send_to_group', array(
                    'group_id'=>$group_id,
                    'message'=>$data['message']
                ));
                break;
        }
    };
    // 여기서 중요한 점은 연결이 닫힐 때 전역 그룹 데이터에서 연결을 삭제하여 메모리 누수를 방지하는 것
    $worker->onClose = function(TcpConnection $con){
        global $group_con_map;
        // 연결이 가입한 모든 그룹을 반복하여 해당 그룹의 데이터를 group_con_map에서 삭제
        if (isset($con->group_id)) {
            foreach ($con->group_id as $group_id) {
                unset($group_con_map[$group_id][$con->id]);
                if (empty($group_con_map[$group_id])) {
                    unset($group_con_map[$group_id]);
                }
            }
        }
    };

    Worker::runAll();
```

## 테스트 (모두 127.0.0.1에서 실행된다고 가정)
1. 서버 실행
``` 
php start.php start
Workerman[del.php] start in DEBUG mode
----------------------- WORKERMAN -----------------------------
Workerman version:3.4.2          PHP version:7.1.3
------------------------ WORKERS -------------------------------
user          worker         listen                    processes status
liliang       ChannelServer  frame://0.0.0.0:2206       1         [OK] 
liliang       none           websocket://0.0.0.0:1234   12        [OK] 
----------------------------------------------------------------
Press Ctrl-C to quit. Start success.
```

2. 클라이언트가 서버에 연결

크롬 브라우저를 열고 F12를 눌러 디버그 콘솔을 열고, Console 탭에서 다음을 입력 (또는 아래 코드를 HTML 페이지에 넣고 JavaScript로 실행)

```javascript
// 서버의 IP가 127.0.0.1이라고 가정하고, 테스트할 때는 실제 서버 IP로 변경
ws = new WebSocket('ws://127.0.0.1:1234');
ws.onmessage = function(data){console.log(data.data)};
ws.onopen = function() {
	ws.send('{"cmd":"add_group", "group_id":"123"}');
    ws.send('{"cmd":"send_to_group", "group_id":"123", "message":"이것은 메시지입니다"}');
};
```
