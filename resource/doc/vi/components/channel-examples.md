# Ví dụ 1
**``` (Yêu cầu phiên bản Workerman >= 3.3.0) ```**

Hệ thống phân phối đa tiến trình (phân tán) dựa trên Worker, gửi cụm, broadcast cụm.

`start_channel.php`
Chỉ có thể triển khai một dịch vụ start_channel cho toàn bộ hệ thống. Giả sử nó đang chạy trên 192.168.1.1.
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Khởi tạo một máy chủ dịch vụ Kênh
$channel_server = new Channel\Server('0.0.0.0', 2206);

Worker::runAll();
```

`start_ws.php`
Hệ thống có thể triển khai nhiều dịch vụ start_ws, giả sử đang chạy trên hai máy chủ 192.168.1.2 và 192.168.1.3.
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Máy chủ websocket
$worker = new Worker('websocket://0.0.0.0:4236');
$worker->count=2;
$worker->name = 'pusher';
$worker->onWorkerStart = function($worker)
{
    // Kết nối từ client đến máy chủ kênh
    Channel\Client::connect('192.168.1.1', 2206);
    // Sử dụng id tiến trình của chính nó làm tên sự kiện
    $event_name = $worker->id;
    // Đăng ký xử lý sự kiện cho việc đăng ký tiến trình->id và tên sự kiện
    Channel\Client::on($event_name, function($event_data)use($worker){
        $to_connection_id = $event_data['to_connection_id'];
        $message = $event_data['content'];
        if(!isset($worker->connections[$to_connection_id]))
        {
            echo "connection not exists\n";
            return;
        }
        $to_connection = $worker->connections[$to_connection_id];
        $to_connection->send($message);
    });

    // Đăng ký sự kiện broadcast
    $event_name = 'Phát sóng';
    // Khi nhận được sự kiện broadcast, gửi dữ liệu broadcast đến tất cả kết nối client trong tiến trình hiện tại
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
Hệ thống có thể triển khai nhiều dịch vụ start_ws, giả sử đang chạy trên hai máy chủ 192.168.1.4 và 192.168.1.5.
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Xử lý yêu cầu http, gửi dữ liệu tới client bất kỳ, yêu cầu workerID và connectionID
$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->name = 'publisher';
$http_worker->onWorkerStart = function()
{
    Channel\Client::connect('192.168.1.1', 2206);
};
$http_worker->onMessage = function(TcpConnection $connection, $request)
{
    // Tương thích với workerman 4.x
    if (!is_array($request)) {
            $_GET = $request->get();
   }
    $connection->send('ok');
    if(empty($_GET['content'])) return;
    // Gửi dữ liệu tới một tiến trình worker cụ thể trong một kết nối cụ thể
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
    // Phát sóng dữ liệu toàn cầu
    else
    {
        $event_name = 'Phát sóng';
        $content = $_GET['content'];
        Channel\Client::publish($event_name, array(
           'content'          => $content
        ));
    }
};

Worker::runAll();
```

## Kiểm tra
1. Chạy dịch vụ trên từng máy chủ
2. Kết nối client với máy chủ
   Mở trình duyệt Chrome, nhấn F12 để mở bảng điều khiển Debug. Trong cửa sổ Console, nhập (hoặc đặt mã dưới đây vào trang html và chạy bằng js)

```javascript
// Cũng có thể kết nối với ws://192.168.1.3:4236
ws = new WebSocket("ws://192.168.1.2:4236");
ws.onmessage = function(e) {
    alert("Nhận thông điện từ máy chủ: " + e.data);
};
```
3. Thông qua giao diện http để gửi
   Truy cập url `http://192.168.1.4:4237/?content={$content}` hoặc `http://192.168.1.5:4237/?content={$content}` để gửi dữ liệu `{$content}` tới tất cả kết nối client.

   Truy cập url `http://192.168.1.4:4237/?to_worker_id={$worker_id}&to_connection_id={$connection_id}&content={$content}` hoặc `http://192.168.1.5:4237/?to_worker_id={$worker_id}&to_connection_id={$connection_id}&content={$content}` để gửi dữ liệu `{$content}` tới một kết nối client cụ thể trong một tiến trình cụ thể.

   Lưu ý: Trong quá trình thử nghiệm, thay thế `{$worker_id}`, `{$connection_id}` và `{$content}` bằng các giá trị thực tế.
