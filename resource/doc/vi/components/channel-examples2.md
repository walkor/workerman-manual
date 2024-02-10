# Ví dụ 1
**``` (Yêu cầu Workerman phiên bản >= 3.3.0) ```**

Hệ thống push đa tiến trình dựa trên Worker

``` php
    use Workerman\Worker;
    use Workerman\Connection\TcpConnection;
    require_once __DIR__ . '/vendor/autoload.php';

    $channel_server = new Channel\Server('0.0.0.0', 2206);

    $worker = new Worker('websocket://0.0.0.0:1234');
    $worker->count = 8;
    // Ánh xạ toàn cầu từ nhóm đến kết nối mảng
    $group_con_map = array();
    $worker->onWorkerStart = function(){
        // Kết nối client Channel tới server Channel
        Channel\Client::connect('127.0.0.1', 2206);

        // Nghe sự kiện gửi tin nhắn nhóm toàn cầu
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
        // Thêm tin nhắn vào nhóm {"cmd":"add_group", "group_id":"123"}
        // Hoặc gửi tin nhắn nhóm {"cmd":"send_to_group", "group_id":"123", "message":"Đây là tin nhắn"}
        $data = json_decode($data, true);
        var_dump($data);
        $cmd = $data['cmd'];
        $group_id = $data['group_id'];
        switch($cmd) {
            // Kết nối tham gia nhóm
            case "add_group":
                global $group_con_map;
                // Thêm kết nối vào mảng nhóm tương ứng
                $group_con_map[$group_id][$con->id] = $con;
                // Ghi lại kết nối đã tham gia nhóm nào, để dễ dàng xóa dữ liệu tương ứng với nhóm trong group_con_map khi đóng
                $con->group_id = isset($con->group_id) ? $con->group_id : array();
                $con->group_id[$group_id] = $group_id;
                break;
            // Gửi tin nhắn đến nhóm
            case "send_to_group":
                // Channel\Client phát tín hiệu gửi tin nhóm đến tất cả các tiến trình trên tất cả các máy chủ
                Channel\Client::publish('send_to_group', array(
                    'group_id'=>$group_id,
                    'message'=>$data['message']
                ));
                break;
        }
    };
    // Quan trọng ở đây, khi kết nối đóng, hãy xóa kết nối khỏi dữ liệu nhóm toàn cầu để tránh rò rỉ bộ nhớ
    $worker->onClose = function(TcpConnection $con){
        global $group_con_map;
        // Duyệt qua tất cả các nhóm mà kết nối đã tham gia, xóa dữ liệu tương ứng từ group_con_map
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

## Kiểm tra (giả sử đều chạy trên máy cục bộ 127.0.0.1)
1. Chạy máy chủ
```bash
php start.php start
Workerman[del.php] start in DEBUG mode
----------------------- WORKERMAN -----------------------------
Workerman version:3.4.2 PHP version:7.1.3
------------------------ WORKERS -------------------------------
user worker         listen                    processes status
liliang ChannelServer  frame://0.0.0.0:2206       1         [OK]
liliang none           websocket://0.0.0.0:1234   12        [OK]
----------------------------------------------------------------
Press Ctrl-C to quit. Start success.
```

2. Kết nối từ phía client
Mở trình duyệt Chrome, nhấn F12 để mở bảng điều khiển và nhập (hoặc chèn đoạn mã dưới đây vào trang html và chạy bằng js)

```javascript
// Giả sử địa chỉ IP máy chủ là 127.0.0.1, khi kiểm tra, vui lòng thay đổi thành địa chỉ IP thực tế của máy chủ
ws = new WebSocket('ws://127.0.0.1:1234');
ws.onmessage = function(data){console.log(data.data)};
ws.onopen = function() {
	ws.send('{"cmd":"add_group", "group_id":"123"}');
    ws.send('{"cmd":"send_to_group", "group_id":"123", "message":"Đây là tin nhắn"}');
};
```
