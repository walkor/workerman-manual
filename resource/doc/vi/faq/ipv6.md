# ipv6

Hỏi: Làm thế nào để khách hàng có thể truy cập bằng cả địa chỉ ipv4 và địa chỉ ipv6?

Trả lời: Khi khởi tạo container, chỉ cần lắng nghe địa chỉ là ```[::]``` là được.

Ví dụ
```php
$worker = new Worker('http://[::]:8080');
$gateway = new Gateway('websocket://[::]:8081');
```
