# สร้าง HTTPS บริการ

**คำถาม:**

Workerman ทำอย่างไรเพื่อสร้างบริการ [https](https://baike.baidu.com/item/https) ซึ่งทำให้ไคลเอนต์สามารถเชื่อมต่อผ่านโพรโทคอล [https](https://baike.baidu.com/item/https) 

**คำตอบ:**

โพรโทคอล [https](https://baike.baidu.com/item/https) ที่ใช้งานจริงแล้วเป็นการผสมระหว่าง[http](https://baike.baidu.com/item/http) และ [SSL](https://baike.baidu.com/item/ssl) คือการเพิ่มชั้น [SSL](https://baike.baidu.com/item/ssl) ที่โจทย์บน[http](https://baike.baidu.com/item/http) Workerman รองรับโพรโทคอล [http](https://baike.baidu.com/item/http) และ รองรับโพรโทคอล [SSL](https://baike.baidu.com/item/ssl) (```ต้องการ Workerman เวอร์ชัน >= 3.3.7```) ดังนั้นเราจะเพียงแค่เปิดใช้งาน [SSL](https://baike.baidu.com/item/ssl)  บนโพรโทคอล [http](https://baike.baidu.com/item/http) เพื่อรองรับโพรโทคอล [https](https://baike.baidu.com/item/https) 

การให้ Workerman รองรับ [https](https://baike.baidu.com/item/https) มีวิธีทำแบบทั่วไปสองวิธี คือ การเปิดใช้งาน SSL โดยตรงบน Workerman และใช้ nginx เป็นตัวแทน SSL 两种方案选其一即可，不可同时设置。

## เปิดใช้งาน SSL บน Workerman

**การเตรียมการ:**

1. Workerman เวอร์ชัน >=3.3.7
2. ติดตั้ง PHP ด้วยการขยาย openssl
3. ได้รับการออกเอกสาร (ไฟล์ pem/crt และไฟล์ key) วางไว้ที่ /etc/nginx/conf.d/ssl

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// ควรใช้เอกสารที่ได้รับการออกเอกสาร
$context = array(
    'ssl' => array(
        'local_cert'        => '/etc/nginx/conf.d/ssl/server.pem', // สามารถก็็ะได้พี่ัอ c ไฟล์
        'local_pk'          => '/etc/nginx/conf.d/ssl/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // ถ้าใช้เอกสารที่ได้รับการออกเอกสารตนเองจำเป็นต้องเปิดตัวเลือกนี้
    )
);
// การตั้งค่าโปรโทคอล http
$worker = new Worker('http://0.0.0.0:443', $context);
// การตั้งค่าการโยกรพทะยานข้อมูลบน ssl เพื่อเป็น http+SSL เป็นไปอย่างไม่สมจัง
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};
Worker::runAll();
```

ผ่าน Workerman ดังกล่าวควสณเปิดใช้งานการบริการ https ทำให้ไคลเอนต์สามารถเชื่อมต่อผ่านโพรโทคอล https เพื่อการสื่อสารที่เข็มันความปลอดภัย

**การทดสอบ:**

กรุณาใส่ชื่อโดเมนลงในแถบที่อยู่ของเบราเซอร์ ```https://domain_name:443``` เพื่อเข้าถึง

**หมายเหตุ:**

1. จำเป็นต้องใช้โพรโทคอล https เพื่อเข้าถึงพอร์ต https	http จึงไม่สามารถเข้าถึง
2. เนื่องจากเอกสารมักจะผูกบั้ันด้งได้อัวงอินเทอร์เน็ต จึงควรใช้โดเมนเมื่อทําการทดสอบขอแนะนําให้ไม่ใช้ไอพีเลําน
3. หากไม่สามารถเข้าถึง https กรุณาตรวจสอบไฟร์วอลห์ของเซินเวอร์ของคุณ

## การใช้ nginx เป็นตัวแทนของ SSL

นอกเหนือจากการใช้ SSL ของ Workerman เอง สามารถใช้ nginx เป็นตัวแทนของ SSL เพื่อให้เป็น https

> **ข้อควรระวัง**
> การใช้งาน nginx เป็นตัวแทนของ SSL และการตั้งค่าใช้งาน SSL ของ Workerman สามารถใช้ได้คู่แค่หนึ่งอย่าให้ใช้ทั้งคู่

หลักการและขั้นตอนการสื่อสารคือ

1. ไคลเอนต์เริ่มต้นการเชื่อต่อ https เพื่อเชื่อมต่อกับ nginx
2. nginx จะแปลงข้อมูลโพรโทคอลเป็น https เป็น http และการส่งต่อไปยังพอร์ท http ของ Workerman
3. Workerman รับข้อมูลและจัดการตามขั้นตอนทางธุ์ะงอก่งไปยังการส่งข้อมูลโพรโทคอล http ให้กับ nginx
4. nginx จะแปลงข้อมูลโพรโทคอล	http เป็น https และส่งไปยังไคลเอนต์

### การตั้งค่า nginx ตัวอย่าง

**เงื่อนไขพื้นฐานและการเตรียมการ:**

1. สมมุติว่า  Workerman ยังการฟังพอร์ต 8181 (โพรโทคอล http)
2. ได้รับเอกสาร (ไฟล์ pem/crt และ key) วางไว้ที่ /etc/nginx/conf.d/ssl และใช้งาน nginx ในการเปิดพอร์ต 443 เพื่อให้สามารถใช้งาน wss พร็อคซี เสริม (พอร์ตสามารถเปลี่ยนแปลงตามความจําเป็นของคุณ)

**การตั้งค่า nginx ตัวอย่างคือดังนี้**：

```nginx
upstream workerman {
    server 127.0.0.1:8181;
    keepalive 10240;
}

server {
  listen 443;
  server_name ชื่อโดเมน.com;
  access_log off;
  
  ssl on;
  ssl_certificate /etc/nginx/conf.d/ssl/server.pem;
  ssl_certificate_key /etc/nginx/conf.d/ssl/server.key;
  ssl_session_timeout 5m;
  ssl_session_cache shared:SSL:50m;
  ssl_protocols SSLv3 SSLv2 TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;

  location / {
    proxy_pass http://workerman;
    proxy_http_version 1.1;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Connection "";
  }
}
```
