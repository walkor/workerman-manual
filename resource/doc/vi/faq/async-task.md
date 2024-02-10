# Cách thực hiện các nhiệm vụ không đồng bộ

**Câu hỏi:**

Làm thế nào để xử lý các nhiệm vụ nặng nề một cách không đồng bộ, tránh việc quá trình chính bị chặn trong khoảng thời gian dài. Ví dụ, tôi muốn gửi email cho 1000 người dùng, quá trình này rất chậm, có thể mất vài giây và quá trình này sẽ ảnh hưởng đến các yêu cầu sau này vì quá trình chính bị chặn, làm thế nào để giao nhiệm vụ nặng nề như vậy cho các quá trình không đồng bộ khác.

**Trả lời:**

Bạn có thể tạo trước một số quá trình xử lý nhiệm vụ nặng nề trên cùng một máy hoặc máy chủ khác hoặc cụm máy chủ, số lượng quá trình nhiệm vụ có thể mở nhiều hơn, ví dụ như là 10 lần CPU, sau đó người gọi sử dụng AsyncTcpConnection để gửi dữ liệu đến những quá trình nhiệm vụ này để xử lý không đồng bộ và nhận kết quả xử lý không đồng bộ.

Máy chủ quá trình nhiệm vụ
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// quá trình nhiệm vụ, sử dụng giao thức Text
$task_worker = new Worker('Text://0.0.0.0:12345');
// số lượng quá trình nhiệm vụ có thể mở ra nhiều hơn tùy theo nhu cầu
$task_worker->count = 100;
$task_worker->name = 'TaskWorker';
$task_worker->onMessage = function(TcpConnection $connection, $task_data)
{
     // Giả sử dữ liệu được gửi đến là dạng json
     $task_data = json_decode($task_data, true);
     // Xử lý nhiệm vụ tương ứng dựa trên task_data.... Lấy kết quả, ở đây ta bỏ qua.....
     $task_result = ......;
     // Gửi kết quả
     $connection->send(json_encode($task_result));
};
Worker::runAll();
```

Gọi trong workerman

```php
use Workerman\Worker;
use \Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// dịch vụ websocket
$worker = new Worker('websocket://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $ws_connection, $message)
{
    // Thiết lập kết nối không đồng bộ với máy chủ nhiệm vụ từ xa, ip là địa chỉ ip của máy chủ nhiệm vụ từ xa, nếu là máy local thì là 127.0.0.1, nếu là cụm máy chủ thì là địa chỉ ip của lvs
    $task_connection = new AsyncTcpConnection('Text://127.0.0.1:12345');
    // Dữ liệu nhiệm vụ và tham số
    $task_data = array(
        'function' => 'send_mail',
        'args'       => array('from'=>'xxx', 'to'=>'xxx', 'contents'=>'xxx'),
    );
    // Gửi dữ liệu
    $task_connection->send(json_encode($task_data));
    // Nhận kết quả không đồng bộ
    $task_connection->onMessage = function(AsyncTcpConnection $task_connection, $task_result)use($ws_connection)
    {
         // Kết quả
         var_dump($task_result);
         // Sau khi nhận kết quả hãy đóng kết nối không đồng bộ
         $task_connection->close();
         // Thông báo cho người dùng websocket tương ứng là nhiệm vụ đã hoàn thành
         $ws_connection->send('task complete');
    };
    // Thực hiện kết nối không đồng bộ
    $task_connection->connect();
};

Worker::runAll();
```

Với cách làm này, nhiệm vụ nặng nề được gửi cho các quá trình trên máy chủ hoặc máy chủ khác để xử lý, sau khi nhiệm vụ hoàn thành, kết quả sẽ được nhận không đồng bộ và quá trình kinh doanh sẽ không bị chặn.
