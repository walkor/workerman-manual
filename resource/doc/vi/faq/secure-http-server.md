# Tạo dịch vụ https

**Câu hỏi:**

Làm thế nào để tạo một dịch vụ [https](https://baike.baidu.com/item/https) trong Workerman, để khách hàng có thể kết nối thông qua giao thức [https](https://baike.baidu.com/item/https).


**Trả lời:**

Thực tế, giao thức [https](https://baike.baidu.com/item/https) là sự kết hợp giữa giao thức [http](https://baike.baidu.com/item/http) và [SSL](https://baike.baidu.com/item/ssl), nghĩa là cở sở của giao thức [http](https://baike.baidu.com/item/http) được bao gồm một lớp [SSL](https://baike.baidu.com/item/ssl). Workerman hỗ trợ giao thức [http](https://baike.baidu.com/item/http), và cùng một lúc cũng hỗ trợ [SSL](https://baike.baidu.com/item/ssl) (```yêu cầu phiên bản Workerman>=3.3.7```), do đó chỉ cần bật [SSL](https://baike.baidu.com/item/ssl) trên cở sở của giao thức [http](https://baike.baidu.com/item/http) để hỗ trợ giao thức [https](https://baike.baidu.com/item/https).

Có hai phương pháp chung để làm cho Workerman hỗ trợ https, một là sử dụng trực tiếp SSL trong Workerman, một là sử dụng nginx làm proxy SSL. Chọn một trong hai phương pháp này và không thể cài đặt cùng một lúc.

## Bật SSL trong Workerman

**Công việc chuẩn bị:**

1. Phiên bản Workerman>=3.3.7
2. Cài đặt PHP đã bao gồm sự mở rộng openssl
3. Đã làm việc với chứng chỉ (tập tin pem/crt và tập tin key) đã được đặt trong /etc/nginx/conf.d/ssl

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Tốt nhất là sử dụng chứng chỉ được cấp phát
$context = array(
    'ssl' => array(
        'local_cert'        => '/etc/nginx/conf.d/ssl/server.pem', // Có thể sử dụng tập tin crt
        'local_pk'          => '/etc/nginx/conf.d/ssl/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // Nếu sử dụng chứng chỉ tự ký, cần bật tùy chọn này
    )
);
// Ở đây làm việc với giao thức http
$worker = new Worker('http://0.0.0.0:443', $context);
// Thiết lập transport để bật ssl, chuyển thành http+SSL tức là https
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

Với đoạn mã trên, Workerman đã tạo dịch vụ https, và khách hàng có thể kết nối với Workerman thông qua giao thức https để thực hiện truyền thông mã hóa an toàn.

**Kiểm tra:**

Nhập địa chỉ trình duyệt ```https://tên miền:443``` để truy cập.

**Lưu ý:**

1. Cổng https phải sử dụng giao thức https, không thể sử dụng giao thức http.
2. Chứng chỉ thường được liên kết với tên miền, vì vậy trong quá trình kiểm tra, vui lòng sử dụng tên miền, không sử dụng địa chỉ IP.
3. Nếu không thể truy cập bằng https, hãy kiểm tra tường lửa máy chủ.


## Sử dụng nginx làm proxy cho SSL

Ngoài việc sử dụng SSL của chính Workerman, bạn cũng có thể sử dụng nginx làm proxy SSL để thực hiện https.

> **Lưu ý**
> Sử dụng nginx làm proxy cho SSL và thiết lập SSL trong Workerman là hai phương pháp tuỳ chọn, không thể mở cả hai cùng một lúc.

Nguyên lý truyền thông và luồng là:

1. Khách hàng khởi tạo kết nối https đến nginx
2. Nginx chuyển đổi dữ liệu giao thức https sang giao thức http và chuyển tiếp đến cổng http của Workerman
3. Workerman nhận dữ liệu và thực hiện xử lý logic kinh doanh, trả về dữ liệu giao thức http cho nginx
4. Nginx chuyển đổi dữ liệu giao thức http sang https và chuyển tiếp đến khách hàng


### Tham chiếu cấu hình nginx
**Điều kiện tiên quyết và công việc chuẩn bị:**

1. Giả sử Workerman đang lắng nghe trên cổng 8181 (giao thức http)
2. Đã làm việc với chứng chỉ (tập tin pem/crt và tập tin key) đã được đặt trong /etc/nginx/conf.d/ssl
3. Định hướng sẵn sàng cho phép cung cấp dịch vụ wss qua cổng 443 (cổng có thể được điều chỉnh theo nhu cầu)

**Cấu hình nginx tương tự như sau**:
```
upstream workerman {
    server 127.0.0.1:8181;
    keepalive 10240;
}


server {
  listen 443;
  server_name domain_name.com;
  access_log off;
  
  ssl on;
  ssl_certificate /etc/nginx/conf.d/ssl/server.pem;
  ssl_certificate_key /etc/nginx/conf.d/ssl/server.key;
  ssl_session_timeout 5m;
  ssl_session_cache shared:SSL:50m;
  ssl_protocols SSLv3 SSLv2 TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;

  location /
  {
    proxy_pass http://workerman;
    proxy_http_version 1.1;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Connection "";
  }
}
```
