# nghe
```php
void Worker::listen(void)
``` 
Được sử dụng để lắng nghe sau khi khởi tạo Worker.

Phương thức này chủ yếu được sử dụng để tạo động một thể hiện Worker mới sau khi Worker tiến trình khởi động, có thể thực hiện lắng nghe nhiều cổng trong cùng một tiến trình và hỗ trợ nhiều giao thức. Cần lưu ý rằng việc sử dụng phương thức này chỉ là để tăng lắng nghe trong tiến trình hiện tại và sẽ không tạo động tiến trình mới và cũng sẽ không gây ra việc kích hoạt phương thức onWorkerStart.

Ví dụ: Sau khi một Worker HTTP được khởi động, thể hiện một Worker websocket, sau đó tiến trình này có thể truy cập thông qua giao thức HTTP và cũng có thể truy cập thông qua giao thức websocket. Vì Worker websocket và Worker HTTP trong cùng một tiến trình, nên chúng có thể truy cập biến bộ nhớ chung và chia sẻ tất cả kết nối socket. Có thể thực hiện việc nhận yêu cầu HTTP, sau đó thao tác với khách hàng websocket hoàn tất công việc gửi dữ liệu đến khách hàng.

**Lưu ý:**

Nếu phiên bản PHP <= 7.0, không hỗ trợ việc tạo động thể hiện Worker của cùng một cổng trong nhiều tiến trình con. Ví dụ, Tiến trình A tạo một Worker lắng nghe cổng 2016, sau đó Tiến trình B sẽ không thể tạo Worker lắng nghe cổng 2016 nữa, nếu không sẽ báo lỗi `Address already in use`. Ví dụ mã sau sẽ "không thể" chạy:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
// 4 cái tiến trình
$worker->count = 4;
// Sau mỗi tiến trình được khởi động, tạo thể hiện Worker mới lắng nghe trong tiến trình hiện tại
$worker->onWorkerStart = function($worker)
{
    /**
     * Khi 4 tiến trình được khởi động, tạo Worker lắng nghe cổng 2016
     * Khi thực thi worker->listen() sẽ báo lỗi Address already in use
     * Nếu worker->count=1 thì sẽ không báo lỗi
     */
    $inner_worker = new Worker('http://0.0.0.0:2016');
    $inner_worker->onMessage = 'on_message';
    // Thực hiện lắng nghe. Sẽ báo lỗi Address already in use ở đây
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

// Chạy worker
Worker::runAll();
```

Nếu phiên bản PHP của bạn >=7.0, bạn có thể thiết lập Worker->reusePort=true, điều này cho phép tạo Worker lắng nghe cùng một cổng trong nhiều tiến trình. Xem ví dụ dưới đây:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2015');
// 4 cái tiến trình
$worker->count = 4;
// Sau mỗi tiến trình được khởi động, tạo thể hiện Worker mới lắng nghe trong tiến trình hiện tại
$worker->onWorkerStart = function($worker)
{
    $inner_worker = new Worker('http://0.0.0.0:2016');
    // Thiết lập tái sử dụng cổng, có thể tạo Worker lắng nghe cùng một cổng (yêu cầu PHP>=7.0)
    $inner_worker->reusePort = true;
    $inner_worker->onMessage = 'on_message';
    // Thực hiện lắng nghe. Lắng nghe bình thường sẽ không báo lỗi
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

// Chạy worker
Worker::runAll();
```

### Ví dụ về server PHP và gửi tin nhắn đến client trong thời gian thực

**Nguyên lý:**

1. Thiết lập một Worker websocket để duy trì kết nối kéo dài với client
2. Worker websocket bên trong thiết lập một Worker text
3. Worker websocket và Worker text là cùng một tiến trình, có thể dễ dàng chia sẻ kết nối client
4. Một hệ thống PHP back-end độc lập nào đó thông qua giao thức text với Worker text
5. Worker text thực hiện việc gửi dữ liệu thông qua kết nối websocket

**Mã và bước:**

push.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Khởi tạo một Worker container, lắng nghe cổng 1234
$worker = new Worker('websocket://0.0.0.0:1234');

/*
 * Lưu ý rằng số tiến trình ở đây phải được thiết lập là 1
 */
$worker->count = 1;
// Khi tiến trình worker khởi động sau đó tạo một Worker text để mở một cổng giao tiếp bên trong
$worker->onWorkerStart = function($worker)
{
    // Mở một cổng bên trong, dễ dàng hệ thống bên trong gửi dữ liệu, Định dạng giao thức văn bản + ký tự xuống dòng
    $inner_text_worker = new Worker('text://0.0.0.0:5678');
    $inner_text_worker->onMessage = function(TcpConnection $connection, $buffer)
    {
        // Mảng $data, có uid, cho biết dữ liệu được đẩy đến trang uid nào đó
        $data = json_decode($buffer, true);
        $uid = $data['uid'];
        // Thông qua workerman, đẩy dữ liệu đến trang uid
        $ret = sendMessageByUid($uid, $buffer);
        // Trả kết quả đẩy
        $connection->send($ret ? 'ok' : 'fail');
    };
    // ## Thực hiện lắng nghe ## 
    $inner_text_worker->listen();
};
// Thêm một thuộc tính mới, dùng để lưu trữ sự tương ứng giữa uid và connection
$worker->uidConnections = array();
// Khi có client gửi tin nhắn
$worker->onMessage = function(TcpConnection $connection, $data)
{
    global $worker;
    // Kiểm tra xem client hiện tại đã được xác minh chưa, tức là đã thiết lập uid chưa
    if(!isset($connection->uid))
    {
       // Chưa xác minh thì lấy gói tin đầu tiên làm uid (ở đây để dễ hiểm thị, không thực hiện xác minh thực sự)
       $connection->uid = $data;
       /* Lưu uid tương ứng với connection, điều này giúp dễ dàng tìm connection thông qua uid,
        * để thực hiện việc đẩy dữ liệu đến uid cụ thể
        */
       $worker->uidConnections[$connection->uid] = $connection;
       return;
    }
};

// Khi một client ngắt kết nối
$worker->onClose = function(TcpConnection $connection)
{
    global $worker;
    if (isset($connection->uid))
    {
        // Xóa sự tương ứng khi kết nối bị ngắt
        unset($worker->uidConnections[$connection->uid]);
    }
};

// Đẩy dữ liệu đến tất cả người dùng đã xác minh
function broadcast($message)
{
   global $worker;
   foreach($worker->uidConnections as $connection)
   {
        $connection->send($message);
   }
}

// Đẩy dữ liệu cho từng uid cụ thể
function sendMessageByUid($uid, $message)
{
    global $worker;
    if (isset($worker->uidConnections[$uid]))
    {
        $connection = $worker->uidConnections[$uid];
        $connection->send($message);
        return true;
    }
    return false;
}

// Chạy tất cả các worker
Worker::runAll();
```

Khởi động dịch vụ sau cùng
 ```php push.php start -d```

Mã js để nhận tin nhắn đẩy đến từ server
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

Mã để đẩy tin nhắn từ server
```php
// Thiết lập kết nối socket tới cổng đẩy bên trong
$client = stream_socket_client('tcp://127.0.0.1:5678', $errno, $errmsg, 1);
// Dữ liệu đẩy, bao gồm trường uid, xác định là đẩy đến uid cụ thể
$data = array('uid'=>'uid1', 'percent'=>'88%');
// Gửi dữ liệu, lưu ý cổng 5678 là cổng giao thức văn bản, giao thức văn bản yêu cầu thêm ký tự xuống dòng ở cuối dữ liệu
fwrite($client, json_encode($data)."\n");
// Đọc kết quả đẩy
echo fread($client, 8192);
```
