# สร้างบริการ wss

**คำถาม:**

Workerman จะสร้างบริการ wss อย่างไรให้ลูกค้าสามารถเชื่อมต่อและสื่อสารโดยใช้ wss protocol เช่นเชื่อมต่อผ่านไว้เซอร์เวอร์ในแอปพลิเคชันเล็กของ มาสเตอร์ตัวเล็กเพื่อเชื่อมต่อไปยังฝั่งเซิร์ฟเวอร์

**คำตอบ:**

wss protocol ทําให้ดูเหมือนกับการทํางานร่วมกันระหว่าง [websocket](https://baike.baidu.com/item/WebSocket) และ [SSL](https://baike.baidu.com/item/ssl) โดยการเพิ่มชั้น [SSL](https://baike.baidu.com/item/ssl) เข้าไปบน [websocket](https://baike.baidu.com/item/WebSocket) protocol เหมือนกับ [https](https://baike.baidu.com/item/https) ([http](https://baike.baidu.com/item/http) + [SSL](https://baike.baidu.com/item/ssl)) ดังนั้นเพียงทํเพิ่ม [SSL](https://baike.baidu.com/item/ssl)  บน [websocket](https://baike.baidu.com/item/WebSocket) protocol จะสนับสนุน wss protocol ได้

## วิธีที่ 1: ใช้ nginx/apache ในการทำ SSL Proxy (แนะนํา)

**หลักการและกระบวนการ**

1. ลูกค้าเปิดการเชื่อมต่อ wss ไปยัง nginx/apache
2. nginx/apache จะเปลี่ยนข้อมูล wss protocol เป็นข้อมูล ws protocol และส่งต่อไปยังพอร์ตของ [websocket](https://baike.baidu.com/item/WebSocket) protocol ของ Workerman
3. Workerman รับข้อมูลและประมวลผลตามกระบวนการบริการ
4. เมื่อ Workerman ส่งข้อมูลมายังลูกค้า ข้อมูลจะถูกเปลี่ยนเป็น wss protocol ผ่าน nginx/apache และส่งถึงลูกค้า

## ตัวอย่างการกําหนดค่าของ nginx
**เงื่อนไขพื้นฐานและการเตรียมไว้สำหรับ:**

1. ได้ทําการติดตั้ง nginx และมีเวอร์ชั่นไม่ตํ่ากว่า 1.3
2. คาดว่า Workerman จะฟังก์ูลที่พอร์ต 8282 (websocket protocol)
3. ได้ขอรับใบรับรอง (ไฟล์ pem/crt และไฟล์ key) และต้องการนํามาไว้ที่ /etc/nginx/conf.d/ssl
4. ต้องการเปิดพอร์ต 443 ให้สามารถใช้งานเพื่อให้บริการ wss ผ่าน nginx (สามารถปรับแก้ได้ตามความต้องการ)
5. ทั่งนี้ นอกจากบริการหลัก อาจต้องใช้บริการอื่นของเว็บไซต์ด้วย ดังนั้นใช้ที่อยู่ ```domain.com/wss``` เป็นทางเข้าของ wss นั้นคือที่อยู่ที่ลูกค้าเชื่อมต่อและบริการ


```nginx
server {
  listen 443;
  # การกําหนด domain ถูกทิ้งไว้...
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
  
  # การกําหนดที่อยู่อื่นของไซต์เว็บ...
}
```

**การทดสอบ**
```javascript
// ใบรับรองจะตรวจสอบชื่อโดเมน กรุณาใช้การเชื่อมต่อในชื่อโดเมน เขียนโดยไม่ใช้อ่านพอร์ต
ws = new WebSocket("wss://domain.com/wss");

ws.onopen = function() {
    alert("เชื่อมต่อสําเร็จ");
    ws.send('tom');
    alert("ส่งข้อความไปยังเซิร์ฟเวอร์: tom");
};
ws.onmessage = function(e) {
    alert("ได้รับข้อความจากเซิร์ฟเวอร์: " + e.data);
};
```

## ใช้ apache ในการทํา wss proxy

สามารถใช้ apache ในการทํา wss proxy เพื่อส่งต่อไปยัง workerman ได้เช่นกัน

เตรียมการ:

1. GatewayWorker ที่ฟังก์ูที่พอร์ต 8282 (websocket protocol)
2. ได้ขอรับใบรับรอง ssl และนํามาไว้ที่ /server/httpd/cert/
3. ใช้ apache ส่งต่อพอร์ต 443 ไปยังพอร์ต 8282
4. httpd-ssl.conf จะถูกโหลดไว้
5. ได้ทําการติดตั้ง OpenSSL ไว้

**เปิดใช้ proxy_wstunnel_module module**
```apache
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_wstunnel_module modules/mod_proxy_wstunnel.so
```

**กําหนด SSL และ ส่งต่อ**
```apache
#extra/httpd-ssl.conf
DocumentRoot "/เว็บไซต์/ไดเรกทอรี่"
ServerName domain

# การกําหนด Proxy Config
SSLProxyEngine on

ProxyRequests Off
ProxyPass /wss ws://127.0.0.1:8282/wss
ProxyPassReverse /wss ws://127.0.0.1:8282/wss

# เพิ่มการรองรับของ SSL Protocol, ลบ Protocol ที่ไม่ปลอดภัย
SSLProtocol all -SSLv2 -SSLv3
# ปรับแต่งชุดการเข้ารหัสดังต่อไปนี้
SSLCipherSuite HIGH:!RC4:!MD5:!aNULL:!eNULL:!NULL:!DH:!EDH:!EXP:+MEDIUM
SSLHonorCipherOrder on
# กําหนะการกําหนดหลักของใบรับรอง
SSLCertificateFile /server/httpd/cert/your.pem
# กําหนดคีย์ส่วนตัวของใบรับรอง
SSLCertificateKeyFile /server/httpd/cert/your.key
# กําหนดไฟล์	เชื่อมต่อของใบรับรอง
SSLCertificateChainFile /server/httpd/cert/chain.pem
```

**การทดสอบ**
```javascript
// ใบรับร้องจะตรวจสอบชื่อโดเมน กรุณาใช้ชื่อโดเมนในการเชื่อมต่อ ระวัง! ไม่ต้องใส่พอร์ต
ws = new WebSocket("wss://domain.com/wss");

ws.onopen = function() {
    alert("เชื่อมต่อสําเร็จ");
    ws.send('tom');
    alert("ส่งข้อความไปยังเซิร์ฟเวอร์: tom");
};
ws.onmessage = function(e) {
    alert("ได้รับข้อความจากเซิร์ฟเวอร์: " + e.data);
};
```
## วิธีการที่สอง, เปิด SSL โดยใช้ Workerman โดยตรง (ไม่แนะนำ)

> **โปรดทราบ**
> ไม่สามารถเปิด SSL ทั้งของ nginx/apache และตั้งค่า SSL ใน Workerman พร้อมกันได้

**การเตรียมการทำงาน:**

1. เวอร์ชั่น Workerman >= 3.3.7
2. ติดตั้ง openssl ใน PHP
3. ได้รับการรับรองจากใบรับรอง (ไฟล์ pem/crt และไฟล์ key) และวางไว้ในไดเร็กทอรีใดก็ได้บนฮาร์ดไดรฟ์

**โค้ด:**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// ดีที่สุดถ้าใช้ใบรับรองที่ได้รับการรับรอง
$context = array(
    // ดูเพิ่มเติมเกี่ยวกับ SSL option ที่ http://php.net/manual/zh/context.ssl.php
    'ssl' => array(
        // โปรดใช้ทางเดียว
        'local_cert'        => 'เส้นทางของไดร์ฟ/server.pem', // สามารถเป็นไฟล์ crt ได้เช่นกัน
        'local_pk'          => 'เส้นทางของไดร์ฟ/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // หากเป็นใบรับรองที่ลงนามเองจะต้องเปิดตัวเลือกนี้
    )
);
// ตั้งค่าโปรโตคอลให้เป็นเว็บซ็อกเก็ต (พอร์ตสามารถเป็นอะไรก็ได้ แต่ต้องรักษาให้ไม่ถูกใช้โดยโปรแกรมอื่น)
$worker = new Worker('websocket://0.0.0.0:8282', $context);
// ตั้งค่า transport เปิดใช้ SSL, websocket+ssl เท่ากับ wss
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

ด้วยโค้ดด้านบน, Workerman ก็จะตรวจจับ wss protocol และ client สามารถเชื่อมต่อกับ Workerman ผ่าน wss protocol เพื่อให้การสื่อสารแบบ real-time เป็นการปลอดภัย

**การทดสอบ**

เปิดเบราว์เซอร์ Chrome, กด F12 เพื่อเปิดคอนโซลของการดีบัค ในส่วน Console พิมพ์เป็นเนื้อหานี้ (หรือนำโค้ดด้านล่างนี้ไปวางใน html แล้วรันด้วย JavaScript)

```javascript
// ใบรับรองจะตรวจสอบชื่อโดเมน กรุณาใช้ชื่อโดเมนเพื่อเชื่อมต่อ, โปรดทราบว่าที่นี่มีพอร์ต
ws = new WebSocket("wss://ชื่อโดเมน.com:8282");
ws.onopen = function() {
    alert("เชื่อต่อสำเร็จ");
    ws.send('tom');
    alert("ส่งข้อความไปที่เซิร์ฟเวอร์: tom");
};
ws.onmessage = function(e) {
    alert("ได้รับข้อความจากเซิร์ฟเวอร์: " + e.data);
};
```

**โปรดทราบ:**

1. หากต้องใช้พอร์ต 443, โปรดใช้วิธีการที่หนึ่งด้วย nginx/apache ในการเปิด wss
2. พอร์ต wss สามารถเข้าถึงได้เฉพาะด้วย wss protocol, ws จะไม่สามารถเข้าถึงพอร์ต wss ได้
3. ใบรับรองทั่วไปจะถูกผูกกับชื่อโดเมนดังนั้นโปรดใช้ชื่อโดเมนเมื่อทดสอบแทนที่จะใช่ ip
4. หากพบปัญหาในการเข้าถึงโปรดตรวจสอบไฟร์วอลของเซิร์ฟเวอร์
5. วิธีการนี้ต้องการ PHP เวอร์ชั่น >= 5.6 เนื่องจากกลุ่มซอฟต์แวร์สำหรับโทรศัพท์มือถือของ WeChat ที่ต้องการ tls1.2 และ PHP ต่ำกว่า 5.6 ไม่รองรับ tls1.2
