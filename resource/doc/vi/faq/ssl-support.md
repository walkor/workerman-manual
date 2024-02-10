# Mã hóa và truyền tải an toàn - SSL/TLS

**Câu hỏi:**

Làm cách nào để đảm bảo an toàn giao tiếp giữa webman và Workerman?

**Trả lời:**

Cách tiện lợi nhất là tạo một lớp mã hóa [SSL](https://baike.baidu.com/item/ssl) trên giao thức truyền tải, ví dụ như giao thức wss, [https](https://baike.baidu.com/item/https) đều được truyền tải dựa trên mã hóa của [SSL](https://baike.baidu.com/item/ssl), rất an toàn. Workerman tự hỗ trợ [SSL](https://baike.baidu.com/item/ssl) (```cần Workerman>=3.3.7```), bạn chỉ cần thiết lập một số thuộc tính để kích hoạt SSL.

Tất nhiên, nhà phát triển cũng có thể thiết lập một cơ chế mã hóa riêng dựa trên một số thuật toán mã hóa cụ thể.

## Cách kích hoạt ssl của Workerman như sau:


**Công việc chuẩn bị:**

1. Phiên bản Workerman không nhỏ hơn 3.3.7
2. Cài đặt extension openssl cho PHP
3. Đã yêu cầu chứng chỉ (tệp pem/crt và tệp key) và đặt chúng trong /etc/nginx/conf.d/ssl

**Mã:**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Động cơ tốt nhất là yêu cầu chứng chỉ
$context = array(
    'ssl' => array(
        'local_cert'        => '/etc/nginx/conf.d/ssl/server.pem', // Có thể là tệp crt
        'local_pk'          => '/etc/nginx/conf.d/ssl/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // Nếu có chứng chỉ tự ký, cần kích hoạt tùy chọn này
    )
);
// Ở đây là cài đặt giao thức websocket, cũng có thể là giao thức http hoặc giao thức khác
$worker = new Worker('websocket://0.0.0.0:443', $context);
// Thiết lập transport để kích hoạt ssl
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```


## Kích hoạt SNI (Server Name Indication) của Workerman để có thể trực tiếp trên cùng một IP và cổng, liên kết nhiều chứng chỉ.


**Hợp nhất tệp.pem và tệp .key:**

Hợp nhất nội dung của tệp .pem và tệp .key tương ứng cho mỗi chứng chỉ, thêm nội dung của tệp .key vào cuối tệp .pem. (Nếu tệp .pem đã chứa khóa riêng tư, bạn có thể bỏ qua bước này.)

**Lưu ý rằng chỉ là một chứng chỉ, không phải copy tất cả chứng chỉ sang một tệp.**

Ví dụ: Sau khi hợp nhất tệp.pem của host1.com, nội dung tệp .pem có thể sẽ như sau:
```text
-----BEGIN CERTIFICATE-----
MIIGXTCBA...
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
MIIFBzCCA...
-----END CERTIFICATE-----
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAA....
-----END RSA PRIVATE KEY-----
```


**Mã:**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$context = array(
    'ssl' => array(
        'SNI_enabled' => true, // Kích hoạt SNI
        'SNI_server_certs' => [ // Thiết lập nhiều chứng chỉ
            'host1.com' => '/path/host1.com.pem', // Chứng chỉ 1
            'host2.com' => '/path/host2.com.pem', // Chứng chỉ 2
        ],
        'local_cert' => '/path/default.com.pem', // Chứng chỉ mặc định
        'local_pk'   => '/path/default.com.key',
    )
);
$worker = new Worker('websocket://0.0.0.0:443', $context);
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```
