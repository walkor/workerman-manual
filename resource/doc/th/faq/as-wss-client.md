# ใช้ Workerman เป็นตัวเชื่อมต่อเป็น ws/wss แบบลูกค้า

บางครั้งต้องการให้ Workerman เป็นลูกค้าที่เชื่อมต่อกับเซิร์ฟเวอร์ผ่านโปรโตคอล ws/wss และทำการแลกเปลี่ยนข้อมูลกัน
ด้านล่างนี้คือตัวอย่าง

## Workerman เป็นลูกค้า ws

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){

    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    
    // เมื่อเชื่อมต่อ WebSocket สำเร็จ
    $con->onWebSocketConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    // เมื่อได้รับข้อความ
    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## Workerman เป็นลูกค้า wss(ws+ssl)

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // ssl ต้องเชื่อมต่อพอร์ต 443
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443');

    // ตั้งค่าการเชื่อมต่อด้วย ssl เพื่อเป็น wss
    $con->transport = 'ssl';

    $con->onWebSocketConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## Workerman เป็นลูกค้า wss(ws+ssl) พร้อมใบรับรอง SSL ท้องถิ่น

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // ตั้งค่าการเข้าถึงเทสต์โฮสต์ของคูณและพอร์ตท้องถิ่นรวมถึงใบรับรอง SSL
    $context_option = array(
        // ตัวเลือก ssl ดูที่ http://php.net/manual/zh/context.ssl.php
        'ssl' => array(
            // ที่อยู่ของใบรับรองท้องถิ่น จะต้องเป็นรูปแบบ PEM และประกอบด้วยใบรับรองและคีย์ส่วนตัวท้องถิ่น
            'local_cert'        => '/your/path/to/pemfile',
            // รหัสผ่านของไฟล์ local_cert
            'passphrase'        => 'your_pem_passphrase',
            // ยินยอมใบรับรองที่ตั้งด้วยตนเอง
            'allow_self_signed' => true,
            // ต้องการตรวจสอบใบรับรอง SSL หรือไม่
            'verify_peer'       => false
        )
    );

    // ssl ต้องเชื่อมต่อพอร์ต 443
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443', $context_option);

    // ตั้งค่าการเชื่อมต่อด้วย ssl เพื่อเป็น wss
    $con->transport = 'ssl';

    $con->onWebSocketConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## การตั้งค่าอื่นๆ

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Protocols\Ws;
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;

$worker = new Worker();
// เมื่อโปรแกรมเริ่มทำงาน
$worker->onWorkerStart = function()
{
    // เชื่อมต่อเซิร์ฟเวอร์ websocket ระยะห่างทางเวลา 55 วินาที
    $ws_connection = new AsyncTcpConnection("ws://127.0.0.1:1234");
    $ws_connection->websocketPingInterval = 55;
    // ส่ง header แบบกำหนดเอง
    $ws_connection->headers = ['token' => 'value'];
    // ตั้งค่าประเภทของข้อมูลเป็น BINARY_TYPE_BLOB เป็นค่าเริ่มต้น
    $ws_connection->websocketType = Ws::BINARY_TYPE_BLOB;
    // เมื่อทำการสามสัญญา TCP เสร็จสิ้น
    $ws_connection->onConnect = function($connection){
        echo "tcp connected\n";
    };
    // เมื่อทำการสามสัญญา websocket เสร็จสิ้น
    $ws_connection->onWebSocketConnect = function(AsyncTcpConnection $con, $response) {
        echo $response;
        $con->send('hello');
    };
    // เมื่อได้รับข้อความจากเซิร์ฟเวอร์ websocket
    $ws_connection->onMessage = function($connection, $data){
        echo "recv: $data\n";
    };
    // เมื่อเกิดข้อผิดพลาดในการเชื่อมต่อ ซึ่งชนิดการเชื่อมต่อไปยังเซิร์ฟเวอร์ websocket ล้มเหลว
    $ws_connection->onError = function($connection, $code, $msg){
        echo "error: $msg\n";
    };
    // เมื่อเชื่อมต่อไปยังเซิร์ฟเวอร์ websocket หลุดออก
    $ws_connection->onClose = function($connection){
        echo "connection closed and try to reconnect\n";
        // หากเชื่อมต่อหลุด ให้ลองเชื่อมต่อใหม่ใน 1 วินาที
        $connection->reConnect(1);
    };
    // หลังจากตั้งค่าทุกอย่างเสร็จเรียบร้อย ทำการเชื่อมต่อ
    $ws_connection->connect();
};
Worker::runAll();
```
