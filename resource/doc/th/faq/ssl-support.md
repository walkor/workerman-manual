# การเข้ารหัสข้อมูลการส่ง-ssl/tls

**คำถาม:**
วิธีการที่แน่ใจว่าการสื่อสารระหว่าง webman และ Workerman เป็นอย่างปลอดภัย?

**คำตอบ:**
วิธีการที่สะดวกที่สุดคือการเพิ่มชั้นการเข้ารหัส [SSL (Secure Sockets Layer)](https://baike.baidu.com/item/ssl) บนโปรโตคอลการสื่อสาร เช่น wss และโพรโตคอล [https](https://baike.baidu.com/item/https) ทั้งหมดใช้การส่งข้อมูลที่เข้ารหัสด้วย [SSL](https://baike.baidu.com/item/ssl) ซึ่งมีความปลอดภัยมาก Workerman รองรับ [SSL](https://baike.baidu.com/item/ssl) (```ต้องการ Workerman >= 3.3.7```) โดยตั้งค่าเพียงเล็กน้อยเท่านั้นและสามารถเปิดใช้งาน SSL ได้

แน่นอนวิธีการสร้างระบบการเข้ารหัสและถอดรหัสขึ้นอยู่กับนักพัฒนาเอง

## วิธีการเปิดใช้งาน SSL ใน Workerman ดังนี้:

**การเตรียมพร้อม:**

1. Workerman เวอร์ชันต้องไม่ต่ำกว่า 3.3.7
2. ติดตั้ง PHP openssl extension
3. ได้รับใบรับรอง (ไฟล์ pem/crt และไฟล์ key) โดยเก็บไว้ที่ /etc/nginx/conf.d/ssl

**โค้ด:**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// ขอแนะนำให้ใช้ใบรับรองจริง
$context = array(
    'ssl' => array(
        'local_cert'        => '/etc/nginx/conf.d/ssl/server.pem', // สามารถใช้ไฟล์ crt ได้
        'local_pk'          => '/etc/nginx/conf.d/ssl/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // หากเป็นใบรับรองที่ลงชื่อเองจะต้องเปิดตัวเลือกนี้
    )
);
// ตรงนี้เป็นการตั้งค่า websocket สามารถใช้ http หรือโพรโตคอลอื่น ๆ ได้เช่นกัน
$worker = new Worker('websocket://0.0.0.0:443', $context);
// ตั้งค่า transport เพื่อเปิดใช้งาน ssl
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
``` 

## เปิดใช้งาน Server Name Indication [SNI (Server Name Indication)](https://baike.baidu.com/item/%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%90%8D%E7%A7%B0%E6%8C%87%E7%A4%BA)

สามารถให้ผู้ใช้ใช้งานหลายใบรับรองบน IP และพอร์ตเดียวกันได้

**รวมไฟล์ pem และ key ของใบรับรอง:**

รวมเนื้อหาของไฟล์ pem และ key ของทุกใบรับรองโดยเพิ่มเนื้อหาของไฟล์ key ลงท้ายของไฟล์ pem (หากไฟล์ pem มีส่วนตัวแล้วสามารถข้ามได้)

**โปรดทราบว่าเราต้องการใบรับรองเพียงใบรับรองเดียว ไม่ใช่การคัดลอกทุกใบรับรองไปยังไฟล์เดียวกัน**

ตัวอย่างเช่นไฟล์ pem ของ *host1.com.pem* กรณีที่รวมแล้วจะมีเนื้อหาประมาณนี้:
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

**โค้ด:**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$context = array(
    'ssl' => array(
        'SNI_enabled' => true, // เปิดใช้งาน SNI
        'SNI_server_certs' => [ // ตั้งค่าใบรับรองหลายใบ
            'host1.com' => '/path/host1.com.pem', // ใบรับรองที่ 1
            'host2.com' => '/path/host2.com.pem', // ใบรับรองที่ 2
        ],
        'local_cert' => '/path/default.com.pem', // ใบรับรองเริ่มต้น
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
