# Example 1
**``` (Workerman 版本要求 >=3.3.0) ```**

基於 Worker 的多進程分組推送系統

``` php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$channel_server = new Channel\Server('0.0.0.0', 2206);

$worker = new Worker('websocket://0.0.0.0:1234');
$worker->count = 8;
// 全局群組到連接的映射數組
$group_con_map = array();
$worker->onWorkerStart = function(){
    // Channel 客戶端連接到 Channel 服務端
    Channel\Client::connect('127.0.0.1', 2206);

    // 監聽全局分組發送消息事件
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
    // 加入群組消息{"cmd":"add_group", "group_id":"123"}
    // 或者 群發消息{"cmd":"send_to_group", "group_id":"123", "message":"這個是消息"}
    $data = json_decode($data, true);
    var_dump($data);
    $cmd = $data['cmd'];
    $group_id = $data['group_id'];
    switch($cmd) {
        // 連接加入群組
        case "add_group":
            global $group_con_map;
            // 將連接加入到對應的群組數組裡
            $group_con_map[$group_id][$con->id] = $con;
            // 記錄這個連接加入了哪些群組，方便在 onclose 的時候清理 group_con_map 對應群組的數據
            $con->group_id = isset($con->group_id) ? $con->group_id : array();
            $con->group_id[$group_id] = $group_id;
            break;
        // 群發消息給群組
        case "send_to_group":
            // Channel\Client 給所有伺服器的所有進程廣播分組發送消息事件
            Channel\Client::publish('send_to_group', array(
                'group_id' => $group_id,
                'message'  => $data['message']
            ));
            break;
    }
};
// 這裡很重要，連接關閉時把連接從全局群組數據中刪除，避免內存洩漏
$worker->onClose = function(TcpConnection $con){
    global $group_con_map;
    // 遍歷連接加入的所有群組，從 group_con_map 刪除對應的數據
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


## 測試 （假設都是本機127.0.0.1運行）
1、運行服務端
```php start.php start
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


2、客戶端連接服務端

打開chrome瀏覽器，按F12打開調試控制台，在Console一欄輸入(或者把下面代碼放入到html頁面用js運行)

```javascript
// 假設服務端ip為127.0.0.1，測試時請改成實際服務端ip
ws = new WebSocket('ws://127.0.0.1:1234');
ws.onmessage = function(data){console.log(data.data)};
ws.onopen = function() {
    ws.send('{"cmd":"add_group", "group_id":"123"}');
    ws.send('{"cmd":"send_to_group", "group_id":"123", "message":"這個是消息"}');
};
```
