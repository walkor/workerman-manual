## วิธีรับหมายเลข IP จริงของผู้ใช้จาก nginx/apache ใน workerman
การใช้ nginx/apache เป็นตัวแทนของ workerman ทำให้ nginx/apache เป็นเสมือนลูกค้าของ workerman ดังนั้น IP ของลูกค้าที่ได้รับจาก workerman คือ IP ของเซิร์ฟเวอร์ nginx/apache แทนที่จะเป็น IP จริงของลูกค้า วิธีการรับหมายเลข IP จริงของผู้ใช้สามารถดูได้จากวิธีต่อไปนี้

**หลักการ:**

nginx/apache จะส่งหมายเลข IP จริงของลูกค้าผ่าน header http เช่น การตั้งค่าใน location ของ nginx โดยมีการเพิ่ม ```proxy_set_header X-Real-IP $remote_addr;``` workerman จะอ่านค่า header นี้และเก็บค่านี้ไว้ใน ```$connection object``` (GatewayWorker สามารถเก็บไว้ในตัวแปร ```$_SESSION```) และสามารถเรียกใช้ค่านี้ได้โดยตรงเมื่อต้องการใช้

**หมายเลขโปรโทคอลที่ใช้:**

การตั้งค่าต่อไปนี้ใช้สำหรับโปรโตคอล http/https ws/wss เท่านั้น สำหรับโปรโตคอลอื่นๆ การรับหมายเลข IP จริงของลูกค้าจะทำได้ตามหลักการเดียวกัน ซึ่งต้องการให้เซิร์ฟเวอร์ตัวแทนเพิ่มหมายเลข IP ลงไปในข้อมูลแพ็คเก็จ

**การตั้งค่า nginx อย่างน้อยตามนี้**:
```
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
    # ส่วนนี้เป็นการส่งหมายเลข IP จริงของลูกค้าผ่าน header http
    proxy_set_header X-Real-IP $remote_addr;
  }
  
  # location / {} การตั้งค่าเว็บไซต์อื่นๆ ...
}
```

**การอ่านหมายเลข IP จริงของผู้ใช้จาก header ที่ตั้งใน nginx**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:7272');

// เมื่อลูกค้าเชื่อมต่อเข้ามา หลังเสร็จสิ้นการต่อ TCP 3 ครั้ง จะมีการเรียกใช้ callback ต่อไปนี้
$worker->onConnect = function(TcpConnection $connection) {
   /**
    * เมื่อทำการ handshake แบบ websocket กับลูกค้า สามารถได้รับค่า X_REAL_IP ที่ถูกส่งมาจาก nginx ผ่าน header http
    */
   $connection->onWebSocketConnect = function(TcpConnection $connection){
       /**
        * ออบเจค connection จริง ๆ ไม่มี property realIP ซึ่งในส่วนนี้จะให้ออบเจค connection เพิ่ม property realIP
        * จำไว้ว่าใน php สามารถเพิ่ม property ให้กับออบเจคได้ และสามารถตั้งชื่อ property ให้กับไอบเจคได้ตามต้องการ
        */
       $connection->realIP = $_SERVER['HTTP_X_REAL_IP'];
   };
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // เมื่อต้องการใช้หมายเลข IP จริงของคลายต้น สามารถใช้ $connection->realIP ได้ทันที
    $connection->send($connection->realIP);
};
Worker::runAll();
```

**GatewayWorker การรับหมายเลข IP จริงของผู้ใช้จาก header ที่ตั้งใน nginx**

แก้ไขไฟล์ Events.php ตามโค้ดต่อไปนี้
```php
class Events
{
   public static function onWebsocketConnect($client_id, $data)
   {    
        $_SESSION['realIP'] = $data['server']['HTTP_X_REAL_IP'];
   }
   // .... ข้ามโค้ดอื่นๆ ....
}
```
เมื่อทำการเปลี่ยนแล้วจำเป็นต้องรีบูต GatewayWorker ใหม่

จากนี้จะสามารถใช้ `$_SESSION['realIP']` ใน `onMessage` และ `onClose` ใน Events.php เพื่อประมาณหมายเลข IP จริงของผู้ใช้ได้

> **หมายเหตุ:** `onWorkerStart` `onConnect` `onWorkerStop` ใน Events.php ไม่สามารถใช้ `$_SESSION['realIP']` โดยตรงได้
