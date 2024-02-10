# Không thể khởi động được workerman

## Hiện tượng 1
Sau khi khởi động, thông báo lỗi tương tự như sau:
```php
php start.php start
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxxx (Address already in use) in ...workerman/Worker.php on line xxxx

```
**Từ khóa quan trọng**: ```Address already in use```

**Nguyên nhân cơ bản**: Cổng đã được sử dụng, không thể khởi động.

#### Giải pháp 1

Bạn có thể sử dụng lệnh ```netstat -anp | grep số_cổng``` để tìm ra chương trình nào đang sử dụng cổng đó. Sau đó dừng chương trình tương ứng để giải phóng cổng.

#### Giải pháp 2
Nếu không thể dừng chương trình sử dụng cổng tương ứng, bạn có thể thay đổi cổng của workerman để giải quyết vấn đề.

#### Giải pháp 3
Nếu cổng được sử dụng bởi Workerman và không thể dừng bằng lệnh stop (thường là do mất tập tin pid hoặc quá trình chính bị kill bởi người phát triển), bạn có thể dùng hai lệnh sau để kết thúc tiến trình Workerman.
```
killall php
ps aux|grep WorkerMan|awk '{print $2}'|xargs kill -9
```

#### Giải pháp 4
Nếu thực sự không có chương trình nào đang nghe cổng đó, có thể là do người phát triển đã thiết lập hai hoặc nhiều lắng nghe trong workerman và các cổng lắng nghe là giống nhau, hãy kiểm tra kịch bản khởi động xem có lắng nghe trên cùng một cổng không.

#### Giải pháp 5
Kiểm tra xem chương trình có kích hoạt reusePort không, hãy thử tắt reusePort.

## Hiện tượng 2
Sau khi khởi động, thông báo lỗi tương tự như sau:
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxx (Cannot assign requested address) in ...workerman/Worker.php on line xxxx
```
hoặc
```
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxxx (在其上下文中，该请求的地址无效) in ...workerman/Worker.php on line xxxx
```
**Từ khóa**: `Cannot assign requested address` hoặc `该请求的地址无效`

**Nguyên nhân thất bại**: 
Tham số ip mà tập lệnh khởi động lắng nghe viết sai, không phải là ip máy chủ, hãy điền đúng ip máy chủ hoặc điền ```0.0.0.0``` (đại diện cho việc lắng nghe tất cả các ip máy chủ của bạn) để giải quyết vấn đề.

**Ghi chú**: Trên hệ điều hành Linux, bạn có thể sử dụng lệnh ```ifconfig``` để xem tất cả các địa chỉ IP của máy chủ. Nếu bạn là người dùng máy chủ đám mây (ví dụ: Alibaba Cloud, Tencent Cloud, v.v.) hãy chú ý rằng địa chỉ IP công cộng thực tế của bạn có thể là địa chỉ IP ủy quyền (ví dụ: mạng riêng của Alibaba), địa chỉ IP công cộng không thuộc về máy chủ hiện tại, do đó không thể lắng nghe thông qua địa chỉ IP công cộng. Mặc dù không thể lắng nghe thông qua địa chỉ IP công cộng, nhưng vẫn có thể sử dụng ```0.0.0.0``` để ràng buộc.

## Hiện tượng 3
```php
Waring stream_socket_server has been disabled for security reasons in ...
```
**Nguyên nhân thất bại**: 
Hàm stream_socket_server bị tắt trong php.ini

**Giải pháp**

1. Chạy lệnh ```php --ini``` để tìm tệp php.ini
2. Mở php.ini và tìm disable_functions, xóa hạng mục cấm sử dụng của stream_socket_server

## Hiện tượng 4
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://0.0.0.0:xxx (Permission denied)
```
**Nguyên nhân thất bại**: 
Trên hệ thống Linux, nếu cổng lắng nghe nhỏ hơn 1024, cần quyền root.

**Giải pháp**

Sử dụng cổng lớn hơn 1024 hoặc sử dụng người dùng root để khởi động dịch vụ.
