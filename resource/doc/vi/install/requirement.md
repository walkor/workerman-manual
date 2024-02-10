# Yêu cầu về môi trường

## Người dùng Windows
Từ phiên bản 3.5.3 trở đi, workerman đã có thể hỗ trợ cùng lúc cả hệ thống linux và windows.

1. Yêu cầu PHP>=5.4 và thiết lập biến môi trường PHP.

2. Windows version của Workerman không yêu cầu cài đặt bất kỳ extension nào.

3. Hướng dẫn cài đặt và giới hạn sử dụng có thể được tìm thấy [**tại đây**](https://www.workerman.net/windows).

4. Do Workerman có nhiều hạn chế khi sử dụng trên Windows, cho nên đề xuất sử dụng hệ thống Linux trong môi trường sản xuất, và chỉ đề xuất sử dụng hệ thống Windows cho môi trường phát triển.

``` ====Phần bên dưới chỉ dành cho người dùng Linux, người dùng Windows vui lòng bỏ qua. ====```

## Người dùng Linux (bao gồm cả Mac OS)
Người dùng Linux chỉ có thể sử dụng phiên bản Workerman dành cho Linux.

1. Cài đặt PHP>=5.4 và cài đặt các extension pcntl, posix.

2. Đề xuất cài đặt extension event, tuy nhiên không bắt buộc (lưu ý rằng extension event yêu cầu PHP>=5.4).

### Script kiểm tra môi trường trên Linux
Người dùng Linux có thể chạy script sau để kiểm tra xem môi trường cục bộ phù hợp với yêu cầu của WorkerMan hay không:

```curl -Ss https://www.workerman.net/check | php```

Nếu tất cả thông báo trong script đều hiển thị "ok", tức là môi trường chạy WorkerMan đủ điều kiện.

(Lưu ý: Script kiểm tra không kiểm tra extension event, nếu số kết nối đồng thời lớn hơn 1024, đề nghị cài đặt extension event, cách cài đặt xem ở phần tiếp theo)

## Thông tin chi tiết

### Về PHP-CLI

WorkerMan chạy dựa trên [PHP-CLI (PHP Command Line Interface)](https://php.net/manual/zh/features.commandline.php). PHP-CLI là một chương trình thực thi độc lập, không xung đột và không phụ thuộc vào PHP-FPM hoặc MOD-PHP của Apache.

### Về các extension mà WorkerMan phụ thuộc

1. [Extension pcntl](https://cn2.php.net/manual/zh/book.pcntl.php)

Extension pcntl là một extension quan trọng của PHP để điều khiển tiến trình trong môi trường Linux. WorkerMan sử dụng các tính năng như [tạo tiến trình](https://cn2.php.net/manual/zh/function.pcntl-fork.php), [điều khiển tín hiệu](https://cn2.php.net/manual/zh/function.pcntl-signal.php), [định thời](https://cn2.php.net/manual/zh/function.pcntl-alarm.php), [giám sát trạng thái tiến trình](https://cn2.php.net/manual/zh/function.pcntl-waitpid.php). Extension này không được hỗ trợ trên nền tảng Windows.

2. [Extension posix](https://cn2.php.net/manual/zh/book.posix.php)

Extension posix cho phép PHP trên môi trường Linux gọi các giao diện được cung cấp bởi hệ thống thông qua [chuẩn POSIX](https://baike.baidu.com/view/209573.htm). WorkerMan chủ yếu sử dụng các giao diện liên quan để thực hiện tính năng tiến trình phần sử dụng, kiểm soát nhóm người dùng và các tính năng khác. Extension này không được hỗ trợ trên nền tảng Windows.

3. [Extension Event](https://php.net/manual/zh/book.event.php) hoặc [Extension libevent](https://cn2.php.net/manual/en/book.libevent.php)

Extension Event cho phép PHP sử dụng các cơ chế xử lý sự kiện nâng cao như [Epoll](https://baike.baidu.com/view/1385104.htm), Kqueue, từ đó nâng cao hiệu suất sử dụng CPU của WorkerMan khi có số kết nối đồng thời lớn. Đây là tính chuyển cần thiết trong các ứng dụng liên quan đến kết nối đồng thời cao và liên tục. Extension libevent (hoặc extension event) không bắt buộc, nếu không cài đặt, WorkerMan mặc định sử dụng cơ chế xử lý sự kiện Select cơ bản của PHP.

## Cách cài đặt các extension

Xem phần [Cài đặt các extension](../appendices/install-extension.md)
