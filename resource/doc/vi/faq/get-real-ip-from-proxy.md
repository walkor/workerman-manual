## Làm thế nào để lấy địa chỉ IP thật của khách hàng thông qua proxy nginx/apache?

Khi sử dụng nginx/apache làm proxy cho workerman, nginx/apache thực tế đóng vai trò như khách hàng của workerman, do đó địa chỉ IP của khách hàng nhận được từ workerman là địa chỉ IP của máy chủ nginx/apache, không phải là địa chỉ IP thực của khách hàng. Để lấy địa chỉ IP thật của khách hàng, bạn có thể tham khảo các phương pháp sau đây.

**Nguyên lý:**

nginx/apache truyền địa chỉ IP thật của khách hàng qua header http, ví dụ như cấu hình nginx trong phần location thêm ```proxy_set_header X-Real-IP $remote_addr;```. Workerman đọc giá trị header này và lưu giữ nó trong ```đối tượng $connection``` (GatewayWorker có thể lưu giữ trong biến ```$_SESSION```), khi sử dụng, bạn có thể đọc trực tiếp từ biến này.

**Chú ý:**

Cấu hình sau áp dụng cho giao thức http/https ws/wss. Các giao thức khác để lấy địa chỉ IP của khách hàng cũng tương tự, yêu cầu máy chủ proxy chèn một đoạn dữ liệu IP vào gói tin để truyền địa chỉ IP thực của khách hàng.

**Cấu hình nginx tương tự như sau**:
```nginx
server {
  listen 443;

  ssl on;
  ssl_certificate /etc/ssl/server.pem;
  ssl_certificate_key /etc/ssl/server.key;
  ssl_session_timeout 5m;
  ssl_session_cache shared:SSL:50m;
  ssl_protocols SSLv3 SSLv2 TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;

  location /wss
  {
    proxy_pass http://127.0.0.1:8282;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
    # Phần này sử dụng header http để truyền địa chỉ IP thực của khách hàng
    proxy_set_header X-Real-IP $remote_addr;
  }
  
  # location / {} Cấu hình khác của trang web...
}
```

**Workerman đọc địa chỉ IP của khách hàng từ header đã được cài đặt trong nginx**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:7272');

// Khi khách hàng kết nối lên, sau khi hoàn thành bắt tay TCP
$worker->onConnect = function(TcpConnection $connection) {
   /**
    * Callback onWebSocketConnect được gọi khi bắt tay websocket với khách hàng
    * Trong callback onWebSocketConnect, chúng ta lấy giá trị X_REAL_IP từ header HTTP của nginx
    */
   $connection->onWebSocketConnect = function(TcpConnection $connection){
       /**
        * Đối tượng kết nối hàng không có thuộc tính realIP, ở đây chúng ta đang động lực thêm một thuộc tính realIP cho đối tượng kết nối
        * Hãy nhớ rằng bạn có thể đặt tên thuộc tính theo sở thích của mình
        */
       $connection->realIP = $_SERVER['HTTP_X_REAL_IP'];
   };
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Khi sử dụng địa chỉ IP thực của khách hàng, bạn có thể sử dụng trực tiếp $connection->realIP
    $connection->send($connection->realIP);
};
Worker::runAll();
```

**GatewayWorker lấy địa chỉ IP của khách hàng từ header đã cài đặt trong nginx**

Thêm đoạn mã sau vào tệp Events.php
```php
class Events
{
   public static function onWebsocketConnect($client_id, $data)
   {    
        $_SESSION['realIP'] = $data['server']['HTTP_X_REAL_IP'];
   }
   // .... Lược đồ mã khác bị bỏ qua....
}
```
Sau khi thêm mã này, bạn cần khởi động lại GatewayWorker.

Như vậy, bạn có thể lấy địa chỉ IP thực của khách hàng trong các phương thức `onMessage` và `onClose` của tệp Events.php bằng `$_SESSION['realIP']`.

> Chú ý: Các phương thức `onWorkerStart`, `onConnect`, `onWorkerStop` trong Events.php không thể trực tiếp sử dụng `$_SESSION['realIP']`.
