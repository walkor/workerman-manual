# Nhịp tim

Lưu ý: Ứng dụng kết nối lâu dài cần có nhịp tim, nếu không có thể dẫn đến việc kết nối bị cắt bởi nút định tuyến do không giao tiếp trong thời gian dài.

Nhịp tim có hai vai trò chính:

1. Máy khách định kỳ gửi dữ liệu đến máy chủ để ngăn chặn việc kết nối bị đóng bởi tường lửa của một số nút do không có giao tiếp trong thời gian dài.

2. Máy chủ có thể dùng nhịp tim để xác định liệu máy khách có trực tuyến hay không. Nếu máy khách không gửi bất kỳ dữ liệu nào trong thời gian quy định, máy chủ sẽ xem máy khách là đang offline. Điều này có thể phát hiện được sự kiện máy khách offline do tình huống cực đoan (mất điện, mất mạng, v.v.).

Thời gian giữa các nhịp tim được đề xuất: 

Đề xuất máy khách gửi nhịp tim có thời gian nhỏ hơn 60 giây, ví dụ 55 giây.

> Định dạng dữ liệu nhịp tim không có yêu cầu cụ thể, miễn là máy chủ có thể nhận biết được.

## Ví dụ về nhịp tim
```php
<?php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Khoảng cách giữa các nhịp tim là 55 giây
define('HEARTBEAT_TIME', 55);

$worker = new Worker('text://0.0.0.0:1234');

$worker->onMessage = function(TcpConnection $connection, $msg) {
    // Tạm thời thiết lập thuộc tính lastMessageTime cho connection, dùng để ghi lại thời gian nhận tin nhắn lần cuối
    $connection->lastMessageTime = time();
    // Các logic kinh doanh khác...
};

// Sau khi quá trình khởi động, thiết lập một bộ hẹn giờ chạy mỗi 10 giây một lần
$worker->onWorkerStart = function($worker) {
    Timer::add(10, function()use($worker){
        $time_now = time();
        foreach($worker->connections as $connection) {
            // Có thể connection này chưa nhận được tin nhắn nào, thì lastMessageTime được thiết lập thành thời gian hiện tại
            if (empty($connection->lastMessageTime)) {
                $connection->lastMessageTime = $time_now;
                continue;
            }
            // Khoảng thời gian qua kênh thông tin lớn hơn khoảng cách giữa các nhịp tim, máy chủ xem xét máy khách đã offline, đóng kết nối
            if ($time_now - $connection->lastMessageTime > HEARTBEAT_TIME) {
                $connection->close();
            }
        }
    });
};

Worker::runAll();
```

Cấu hình trên sẽ đóng kết nối nếu máy khách không gửi bất kỳ dữ liệu nào cho máy chủ trong hơn 55 giây và gửi sự kiện onClose.

## Kết nối lại sau khi bị ngắt kết nối (Quan trọng)

Dù là máy khách gửi nhịp tim hay máy chủ gửi nhịp tim, kết nối đều có thể bị ngắt. Ví dụ như trình duyệt bị thu nhỏ cửa sổ nên js bị tạm dừng, trình duyệt chuyển sang tab trang khác js bị tạm dừng, máy tính vào chế độ ngủ, di động chuyển mạng, tín hiệu yếu, màn hình điện thoại tắt, ứng dụng điện thoại chuyển sang chế độ nền, sự cố định tuyến, chủ doanh nghiệp ngắt kết nối, v.v. Đặc biệt là môi trường mạng ngoại vi phức tạp, nhiều nút định tuyến sẽ dọn sạch kết nối không hoạt động trong vòng 1 phút, đó cũng là lý do tại sao khoảng cách giữa các nhịp tim được đề xuất là nhỏ hơn 1 phút.

Kết nối dễ bị ngắt trong môi trường mạng ngoại vi, nên việc kết nối lại sau khi bị ngắt là chức năng bắt buộc mà ứng dụng kết nối lâu dài cần có (chức năng kết nối lại sau khi bị ngắt chỉ có thể do máy khách thực hiện, máy chủ không thể thực hiện). Ví dụ như trình duyệt websocket cần lắng nghe sự kiện onclose, khi sự kiện onclose xảy ra, mở kết nối mới (để tránh cần phải tắt và mở lại kết nối một cách giãn cách để tránh sự cố). Thậm chí, máy chủ cũng nên định kỳ gửi dữ liệu nhịp tim và máy khách cần định kỳ kiểm tra xem dữ liệu nhịp tim từ máy chủ có vượt quá thời gian quy định hay không, nếu sau thời gian quy định vẫn chưa nhận được dữ liệu nhịp tim từ máy chủ, máy khách cần coi kết nối đã bị ngắt, thực hiện close và mở kết nối mới.
