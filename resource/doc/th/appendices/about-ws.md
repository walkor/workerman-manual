# โปรโตคอล ws

ขณะนี้เวอร์ชันโปรโตคอลของ Workerman คือ **เวอร์ชัน 13**.

workerman สามารถทำหน้าที่เป็นไคลเอ็นต์ โดยใช้โปรโตคอล ws เพื่อเชื่อมต่อ websocket ไปยังเซิร์ฟเวอร์ websocket ระยะไกลเพื่อให้เกิดการสื่อสารสองทาง

> **โปรดทราบ**
> โปรโตคอล ws นั้นสามารถใช้ AsyncTcpConnection ในการทำงานเป็นไคลเอ็นต์เท่านั้น และไม่สามารถใช้เป็นโปรโตคอลที่ใช้ฟังก์ชันการเข้าฟังการเชื่อมต่อเป็นเซิร์ฟเวอร์ได้ กล่าวคือ การใช้งานดังตัวอย่างต่อไปนี้ คือ ไม่ถูกต้อง

```php
$worker = new Worker('ws://0.0.0.0:8080');
```

หากต้องการใช้ workerman เป็นเซิร์ฟเวอร์ websocket โปรดใช้ [โปรโตคอล websocket](about-websocket.md) แทน

**ตัวอย่างโปรโตคอล ws เป็นไคลเอ็นต์ของ websocket:**

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Protocols\Ws;
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;

$worker = new Worker();
// เมื่อเริ่มกระบวนการ
$worker->onWorkerStart = function()
{
    // เชื่อมต่อไปยังเซิร์ฟเวอร์ websocket ระยะไกล โดยใช้โปรโตคอล websocket
    $ws_connection = new AsyncTcpConnection("ws://127.0.0.1:1234");
    // ส่งซ็อกเก็ต websocket 0x9 ไปยังเซิร์ฟเวอร์ทุก 55 วินาที (ตัวเลือก)
    $ws_connection->websocketPingInterval = 55;
    // กำหนดหัว HTTP (ตัวเลือก)
    $ws_connection->headers = [
        'Cookie' => 'PHPSID=82u98fjhakfusuanfnahfi; token=2hf9a929jhfihaf9i',
        'OtherKey' => 'values'
    ];
    // กำหนดประเภทของข้อมูล (ตัวเลือก)
    $ws_connection->websocketType = Ws::BINARY_TYPE_BLOB; // BINARY_TYPE_BLOB สำหรับข้อความ BINARY_TYPE_ARRAYBUFFER สำหรับข้อมูลแบบไบนารี
    // เมื่อ TCP เชื่อมต่อสำเร็จแล้ว (ตัวเลือก)
    $ws_connection->onConnect = function($connection){
        echo "tcp connected\n";
    };
    // เมื่อ websocket เชื่อมต่อสำเร็จแล้ว (ตัวเลือก)
    $ws_connection->onWebSocketConnect = function(AsyncTcpConnection $con, $response) {
        echo $response;
        $con->send('hello');
    };
    // เมื่อมีข้อความจากเซิร์ฟเวอร์ websocket ระยะไกลมาถึง
    $ws_connection->onMessage = function($connection, $data){
        echo "recv: $data\n";
    };
    // เกิดข้อผิดพลาดในการเชื่อมต่อ ซึ่งอาจจะเกิดจากการเชื่อมต่อกับเซิร์ฟเวอร์ websocket ระยะไกลไม่สำเร็จ (ตัวเลือก)
    $ws_connection->onError = function($connection, $code, $msg){
        echo "error: $msg\n";
    };
    // เมื่อเชื่อมต่อกับเซิร์ฟเวอร์ websocket ระยะไกลถูกตัดการเชื่อมต่อ (ตัวเลือก แนะนำให้กระทำการเชื่อมต่ออีกครั้ง)
    $ws_connection->onClose = function($connection){
        echo "connection closed and try to reconnect\n";
        // หากการเชื่อมต่อถูกตัด ให้เชื่อมต่อใหม่ในอีก 1 วินาที
        $connection->reConnect(1);
    };
    // เมื่อตั้งค่า callback ทั้งหมดเรียบร้อย ให้ดำเนินการเชื่อมต่อ
    $ws_connection->connect();
};
Worker::runAll();
```

ดูข้อมูลเพิ่มเติมที่ [เป็นไคลเอ็นต์ ws/wss](../faq/as-wss-client.md)
