# Tạo dịch vụ wss

**Câu hỏi:**

Làm thế nào để tạo ra một dịch vụ wss bằng Workerman để cho phép máy khách kết nối thông qua giao thức wss, ví dụ như trong ứng dụng nhỏ của WeChat kết nối với máy chủ.

**Trả lời:**

Giao thức wss thực tế là [websocket](https://baike.baidu.com/item/WebSocket)+[SSL](https://baike.baidu.com/item/ssl), nghĩa là thêm lớp [SSL](https://baike.baidu.com/item/ssl) vào giao thức websocket, tương tự như [https](https://baike.baidu.com/item/https)([http](https://baike.baidu.com/item/http)+[SSL](https://baike.baidu.com/item/ssl)).
Vì vậy, chỉ cần kích hoạt [SSL](https://baike.baidu.com/item/ssl) trên cơ sở của giao thức [websocket](https://baike.baidu.com/item/WebSocket) là có thể hỗ trợ giao thức wss.

## Phương pháp một, Sử dụng Nginx/Apache để chuyển tiếp SSL (Được khuyến nghị)

**Nguyên lý và quá trình truyền thông**

1. Máy khách khởi tạo kết nối wss đến Nginx/Apache
2. Nginx/Apache chuyển đổi dữ liệu giao thức wss thành dữ liệu giao thức ws và chuyển tiếp đến cổng websocket trong Workerman
3. Workerman nhận được dữ liệu và thực hiện xử lý logic kinh doanh
4. Khi Workerman gửi tin nhắn đến máy khách, quá trình ngược lại xảy ra, dữ liệu được chuyển đổi bằng Nginx/Apache thành giao thức wss và gửi đến máy khách

## Cấu hình Nginx tham khảo

**Điều kiện tiên quyết và chuẩn bị công việc**

1. Đã cài đặt Nginx, phiên bản không thấp hơn 1.3
2. Giả sử Workerman lắng nghe trên cổng 8282 (giao thức websocket)
3. Đã yêu cầu chứng chỉ (tệp pem/crt và tệp key) và giả sử đặt chúng trong /etc/nginx/conf.d/ssl
4. Dự định sử dụng Nginx để kích hoạt cổng 443 đối ngoại để cung cấp dịch vụ wss (cổng có thể được sửa đổi theo nhu cầu)
5. Thông thường, Nginx hoạt động như một máy chủ trang web khác, để không ảnh hưởng đến việc sử dụng trang web ban đầu, ở đây sử dụng địa chỉ ```domain.com/wss``` làm lối vào chuyển tiếp wss. Tức là địa chỉ kết nối của máy khách là wss://domain.com/wss

**Cấu hình Nginx tương tự như sau:**

```nginx
server {
  listen 443;
  # Cấu hình tên miền bị lược bỏ...

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
    proxy_set_header X-Real-IP $remote_addr;
  }
  
  # Cấu hình trang web khác...  
}
```

**Kiểm tra**

```javascript
// Chứng chỉ sẽ kiểm tra tên miền, hãy sử dụng tên miền để kết nối. Chú ý rằng, không cần phải chỉ rõ cổng ở đây
ws = new WebSocket("wss://domain.com/wss");

ws.onopen = function() {
    alert("Kết nối thành công");
    ws.send('tom');
    alert("Gửi một chuỗi tới máy chủ: tom");
};
ws.onmessage = function(e) {
    alert("Nhận được tin nhắn từ máy chủ: " + e.data);
};
```

## Sử dụng Apache để chuyển tiếp wss

Cũng có thể sử dụng Apache như một cách chuyển tiếp wss để chuyển tiếp đến Workerman.

Chuẩn bị công việc:

1. GatewayWorker lắng nghe cổng 8282 (giao thức websocket)
2. Đã yêu cầu chứng chỉ SSL, giả định chúng được đặt trong/sẽver/httpd/cert/
3. Chuyển tiếp cổng 443 từ Apache đến cổng 8282

**Kích hoạt module proxy_wstunnel_module**

```apache
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_wstunnel_module modules/mod_proxy_wstunnel.so
```

**Cấu hình SSL và chuyển tiếp**

```apache
#extra/httpd-ssl.conf
DocumentRoot "/đường dẫn đến thư mục trang web"
ServerName domain

# Proxy Config
SSLProxyEngine on

ProxyRequests Off
ProxyPass /wss ws://127.0.0.1:8282/wss
ProxyPassReverse /wss ws://127.0.0.1:8282/wss

# Thêm hỗ trợ giao thức SSL, loại bỏ các giao thức không an toàn
SSLProtocol all -SSLv2 -SSLv3
# Sửa đổi bộ mã hóa như sau
SSLCipherSuite HIGH:!RC4:!MD5:!aNULL:!eNULL:!NULL:!DH:!EDH:!EXP:+MEDIUM
SSLHonorCipherOrder on
# Cấu hình khóa công cộng của chứng chỉ
SSLCertificateFile /server/httpd/cert/your.pem
# Cấu hình khóa riêng của chứng chỉ
SSLCertificateKeyFile /server/httpd/cert/your.key
# Chuỗi chứng chỉ
SSLCertificateChainFile /server/httpd/cert/chain.pem
```

**Kiểm tra**

```javascript
// Chứng chỉ sẽ kiểm tra tên miền, hãy sử dụng tên miền để kết nối. Chú ý rằng, không cần phải chỉ rõ cổng ở đây
ws = new WebSocket("wss://domain.com/wss");

ws.onopen = function() {
    alert("Kết nối thành công");
    ws.send('tom');
    alert("Gửi một chuỗi tới máy chủ: tom");
};
ws.onmessage = function(e) {
    alert("Nhận được tin nhắn từ máy chủ: " + e.data);
};
```

## Phương pháp hai, Kích hoạt SSL trực tiếp từ Workerman (Không được khuyến nghị)

> **Lưu ý**
> Chỉ có thể kích hoạt SSL từ Nginx/Apache hoặc từ Workerman, không thể kích hoạt cả hai cùng lúc.

**Chuẩn bị công việc:**

1. Phiên bản Workerman>=3.3.7
2. PHP đã cài đặt tiện ích openssl
3. Đã yêu cầu chứng chỉ (tệp pem/crt và tệp key) và giả sử chúng được đặt trong một thư mục bất kỳ trên ổ đĩa

**Mã:**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Chứng chỉ tốt nhất là chứng chỉ đã được yêu cầu
$context = array(
    // Thêm các tùy chọn ssl khác, xem tài liệu hướng dẫn http://php.net/manual/zh/context.ssl.php
    'ssl' => array(
        // Hãy sử dụng đường dẫn tuyệt đối
        'local_cert'        => 'đường_dẫn_ổ_đĩa/server.pem', //Cũng có thể là tệp crt
        'local_pk'          => 'đường_dẫn_ổ_đĩa/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, //Nếu là chứng chỉ tự ký thì cần bật tùy chọn này
    )
);
// Ở đây cài đặt là giao thức websocket (cổng có thể là bất kỳ, nhưng cần đảm bảo không bị chiếm bởi ứng dụng khác)
$worker = new Worker('websocket://0.0.0.0:8282', $context);
// Kích hoạt SSL, websocket+ssl tương đương với wss
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

Với đoạn mã trên, Workerman sẽ lắng nghe giao thức wss, máy khách có thể kết nối với Workerman thông qua giao thức wss để thực hiện trò chuyện trực tuyến an toàn.

**Kiểm tra**

Mở trình duyệt Chrome, nhấn F12 để mở cửa sổ điều khiển gỡ lỗi, trong tab Console nhập (hoặc đặt đoạn mã dưới đây vào trang HTML và chạy bằng JS)

```javascript
// Chứng chỉ sẽ kiểm tra tên miền, hãy sử dụng tên miền để kết nối, chú ý rằng ở đây có cả cổng
ws = new WebSocket("wss://domain.com:8282");
ws.onopen = function() {
    alert("Kết nối thành công");
    ws.send('tom');
    alert("Gửi một chuỗi tới máy chủ: tom");
};
ws.onmessage = function(e) {
    alert("Nhận được tin nhắn từ máy chủ: " + e.data);
};
```

**Lưu ý:**

1. Nếu cần sử dụng cổng 443, vui lòng sử dụng phương pháp đầu tiên, kích hoạt wss thông qua Nginx/Apache.
2. Cổng wss chỉ có thể truy cập qua giao thức wss, không thể truy cập qua cổng ws.
3. Chứng chỉ thường được liên kết với tên miền, vì vậy khi kiểm tra, máy khách vui lòng sử dụng tên miền kết nối, không sử dụng địa chỉ IP.
4. Nếu gặp sự cố về việc truy cập bị từ chối, vui lòng kiểm tra tường lửa máy chủ.
5. Phương pháp này yêu cầu phiên bản PHP >= 5.6 vì ứng dụng nhỏ của WeChat yêu cầu tls1.2, và các phiên bản PHP dưới 5.6 không hỗ trợ tls1.2.

Bài viết liên quan:  
[Lấy địa chỉ IP thật từ proxy](get-real-ip-from-proxy.md)  
[Tùy chọn ngữ cảnh SSL của Workerman tham khảo](https://php.net/manual/zh/context.ssl.php)
