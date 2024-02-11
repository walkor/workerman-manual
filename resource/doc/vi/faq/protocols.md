# Workerman hỗ trợ các giao thức nào

Workerman hỗ trợ đa dạng các giao thức thông qua việc tuân thủ giao diện ```ConnectionInterface``` (xem phần Định custom giao thức). 

Để thuận tiện cho các nhà phát triển, Workerman cung cấp HTTP, WebSocket và giao thức văn bản đơn giản cũng như giao thức frame có thể sử dụng để truyền tải dữ liệu nhị phân. Nhà phát triển có thể sử dụng trực tiếp những giao thức này mà không cần phải phát triển lần thứ hai. Nếu những giao thức này không đáp ứng nhu cầu, nhà phát triển có thể tham khảo phần Định custom giao thức để triển khai giao thức của riêng mình.

Nhà phát triển cũng có thể sử dụng trực tiếp các giao thức tcp hoặc udp.

Ví dụ sử dụng giao thức
```php
// giao thức http
$worker1 = new Worker('http://0.0.0.0:1221');
// giao thức websocket
$worker2 = new Worker('websocket://0.0.0.0:1222');
// giao thức văn bản (giao thức telnet)
$worker3 = new Worker('text://0.0.0.0:1223');
// giao thức frame (có thể sử dụng cho truyền tải dữ liệu nhị phân)
$worker3 = new Worker('frame://0.0.0.0:1223');
// sử dụng trực tiếp giao thức tcp
$worker4 = new Worker('tcp://0.0.0.0:1224');
// sử dụng trực tiếp giao thức udp
$worker5 = new Worker('udp://0.0.0.0:1225');
```
