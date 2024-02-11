# Nguyên lý khởi động lại mượt mà
## Định nghĩa khởi động lại mượt mà là gì?

Khởi động lại mượt mà không giống như việc khởi động lại thông thường, khởi động lại mượt mà có thể được thực hiện mà không ảnh hưởng đến người dùng (thông thường áp dụng cho dịch vụ kết nối ngắn), để tải lại chương trình PHP và hoàn thành cập nhật mã kinh doanh.

Khởi động lại mượt mà thường được áp dụng trong quá trình cập nhật kinh doanh hoặc xuất bản phiên bản, có thể tránh được ảnh hưởng tạm thời của việc làm dịch vụ không sẵn có do khởi động lại dịch vụ khi phát hành mã. 

> **Chú ý**
> Hệ điều hành Windows không hỗ trợ việc tải lại.

> **Chú ý**
> Dịch vụ kết nối dài (ví dụ: WebSocket) sẽ bị ngắt kết nối khi thực hiện khởi động lại mượt mà. Giải pháp là sử dụng kiến trúc giống như [gatewayWorker](https://www.workerman.net/doc/gateway-worker), một nhóm quá trình chuyên trách duy trì kết nối và đặt thuộc tính [reloadable](../worker/reloadable.md) của nhóm quá trình này thành false. Phần logic kinh doanh sẽ khởi động một nhóm quá trình worker khác, truy cập giữa gateway và quá trình worker thông qua giao tiếp TCP. Khi cần thay đổi kinh doanh, chỉ cần khởi động lại quá trình worker là được.

## Giới hạn
**Chú ý: Chỉ các tệp được tải trong các lời gọi lại on{...} mới sẽ được cập nhật tự động sau khi khởi động lại mượt mà, các tệp được tải trực tiếp trong tập lệnh khởi động hoặc mã cứng cố định sẽ không tự động cập nhật khi tải lại.**

#### Đoạn mã sau tải lại không được cập nhật
```php
$worker = new Worker('http://0.0.0.0:1234');
$worker->onMessage = function($connection, $request) {
    $connection->send('Chào'); // Mã cứng cố định không hỗ trợ cập nhật nhanh chóng
};
```

```php
$worker = new Worker('http://0.0.0.0:1234');
require_once __DIR__ . '/your/path/MessageHandler.php'; // Tệp tải trực tiếp trong tập lệnh khởi động không hỗ trợ cập nhật nhanh chóng
$messageHandler = new MessageHandler();
$worker->onMessage = [$messageHandler, 'onMessage']; // Giả sử lớp MessageHandler có một phương thức onMessage
```

#### Đoạn mã sau khi tải lại sẽ tự động cập nhật
```php
$worker = new Worker('http://0.0.0.0:1234');
$worker->onWorkerStart = function($worker) { // onWorkerStart là một lời gọi lại được kích hoạt sau khi quá trình được khởi động
    require_once __DIR__ . '/your/path/MessageHandler.php'; // Tệp được tải sau khi quá trình được khởi động hỗ trợ cập nhật nhanh chóng
    $messageHandler = new MessageHandler();
    $worker->onMessage = [$messageHandler, 'onMessage'];
};
```
Khi sửa đổi MessageHandler.php và thực thi `php start.php reload`, MessageHandler.php sẽ được tải lại vào bộ nhớ để cập nhật logic kinh doanh.

> **Lưu ý**
> Đoạn mã trên sử dụng câu lệnh `require_once` để dễ thể hiện, nếu dự án của bạn hỗ trợ tải tự động theo chuẩn PSR4, thì không cần sử dụng câu lệnh `require_once`.

## Nguyên lý khởi động lại mượt mà

Workerman chia thành quá trình chính và các quá trình con, quá trình chính chịu trách nhiệm theo dõi các quá trình con, quá trình con chịu trách nhiệm nhận kết nối từ khách hàng và xử lý dữ liệu yêu cầu được gửi từ kết nối, sau đó trả dữ liệu cho khách hàng. Khi mã kinh doanh được cập nhật, thực tế chúng ta chỉ cần cập nhật quá trình con để đạt được mục tiêu cập nhật mã.

Khi quá trình chính của Workerman nhận được tín hiệu khởi động lại mượt mà, quá trình chính sẽ gửi tín hiệu thoát an toàn (cho phép quá trình tương ứng hoàn tất xử lý yêu cầu hiện tại trước khi thoát) đến một quá trình con. Khi quá trình này thoát, quá trình chính sẽ tạo một quá trình con mới (quá trình này tải mã PHP mới), sau đó quá trình chính tiếp tục gửi lệnh dừng tới một quá trình cũ khác, theo cách này, mỗi quá trình sẽ được khởi động lại một cách tuần tự, cho đến khi tất cả các quá trình cũ đều được thay thế.

Chúng ta thấy rằng việc khởi động lại mượt mà thực tế là khiến tất cả các quá trình kinh doanh cũ thoát một cách tuần tự và tạo ra từng quá trình mới. Để không ảnh hưởng đến người dùng khi thực hiện khởi động lại mượt mà, điều này đòi hỏi không lưu trạng thái liên quan đến người dùng trong quá trình, có nghĩa là quá trình kinh doanh tốt nhất là không có trạng thái, tránh nguy cơ mất thông tin khi quá trình thoát.
