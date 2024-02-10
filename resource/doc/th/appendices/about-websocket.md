# โปรโตคอล WebSocket

ขณะนี้เวอร์ชันของโปรโตคอล WebSocke ใน Workerman คือเวอร์ชัน 13

โปรโตคอล WebSocket คือโปรโตคอลใหม่ที่เป็นส่วนหนึ่งของ HTML5 ซึ่งมีการทำให้การสื่อสารระหว่างบราวเซอร์และเซิร์ฟเวอร์เป็นการสื่อสารแบบ full-duplex

## ความสัมพันธ์ระหว่าง WebSocket และ TCP

WebSocket เหมือนกับ HTTP ว่าเป็นโปรโตคอลขั้น แอปพลิเคชั่น ทั้งสองนี้จะใช้พื้นฐานการสื่อสารของ TCP แต่ WebSocket จะไม่มีความสัมพันธ์กับ Socket และไม่สามารถเทียบเท่ากันได้

## การ Handshake ของโปรโตคอล WebSocket

โปรโตคอล WebSocket มีขั้นตอนการ Handshake โดยในขณะที่ Handshake บราวเซอร์และเซิร์ฟเวอร์จะสื่อสารด้วยโปรโตคอล HTTP ใน Workerman สามรถเข้าถึงการ Handshake ได้ดังนี้

**เมื่อ workerman <= 4.1**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Connection\TcpConnection;
use Workerman\Worker;

$ws = new Worker('websocket://0.0.0.0:8181');
$ws->onConnect = function($connection)
{
    $connection->onWebSocketConnect = function($connection , $httpBuffer)
    {
        // สามารถทำการตรวจสอบว่าการเชื่อมต่อเป็นไปตามที่ต้องการหรือไม่ หากไม่ตรงตามที่ต้องการก็สามารถปิดการเชื่อมต่อได้
        // $_SERVER['HTTP_ORIGIN'] คือการระบุว่าการเชื่อมต่อ WebSocket เป็นที่มาจากไซต์ใด
        if($_SERVER['HTTP_ORIGIN'] != 'https://www.workerman.net')
        {
            $connection->close();
        }
        // และใน onWebSocketConnect สามารถใช้งาน $_GET และ $_SERVER ได้
        // var_dump($_GET, $_SERVER);
    };
};
Worker::runAll();
```

**เมื่อ workerman >= 5.0**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Worker;

$worker = new Worker('websocket://0.0.0.0:12345');
$worker->onWebSocketConnect = function (TcpConnection $connection, Request $request) {
    if ($request->header('origin') != 'https://www.workerman.net') {
        $connection->close();
    }
    var_dump($request->get());
    var_dump($request->header());
};
Worker::runAll();
```


## การสื่อสารข้อมูลฐานสิทธิ์ของโปรโตคอล WebSocket

โปรโตคอล WebSocket มีค่าเริ่มต้นที่สามารถส่งข้อมูลเป็น utf8 เท็กซ์เท่านั้น หากต้องการส่งข้อมูลฐานสิทธิ์ สามารถทำการอ่านเพิ่มเติมที่ข้อความต่อไปนี้ ในโปรโตคอล WebSocket มีการใช้ flag ในหัวข้อของโปรโตคอลมาให้บ่งบอกว่าข้อมูลที่ถูกส่งไปคือข้อมูลฐานสิทธิ์ หรือ utf8 เท็กซ์ บราวเซอร์จะทำการตรวจสอบ flag และชนิดของข้อมูลที่ถูกส่ง หากไม่ตรงตามจะเกิดข้อผิดพลาดและตัดการเชื่อมต่อ

ดังนั้นการส่งข้อมูลที่เซิร์ฟเวอร์ต้องตั้งค่า flag นี้ตามชนิดของข้อมูลที่ถูกส่ง ใน Workerman หากเป็นข้อมูลธรรมดาแบบ utf8 ควรตั้งค่าเป็นนี้ (ค่าเริ่มต้นมีค่าเป็น utf8 แล้วเช่นกันซึ่งมักจะไม่ต้องตั้งค่าเอง)
```php
use Workerman\Protocols\Websocket;
$connection->websocketType = Websocket::BINARY_TYPE_BLOB;
```

หากเป็นข้อมูลฐานสิทธิ์ ควรตั้งค่าเป็น
```php
use Workerman\Protocols\Websocket;
$connection->websocketType = Websocket::BINARY_TYPE_ARRAYBUFFER;
```

**หมายเหตุ**: หากไม่ได้ทำการตั้งค่า $connection->websocketType ระบบจะเริ่มต้นเป็น BINARY_TYPE_BLOB (ซึ่งคือชนิดของ utf8 เท็กซ์) ในการใช้งานทั่วไปโดยทั่วไปข้อมูลที่ถูกส่งแบบ utf8 เช่นเดียวกับการส่งข้อมูล json ดังนั้นไม่จำเป็นต้องตั้งค่า $connection->websocketType อีก ต้องแค่เมื่อส่งข้อมูลฐานสิทธิ์เท่านั้น (เช่น ข้อมูลรูปภาพ ข้อมูล protobuffer ฯลฯ) ที่จะให้ตั้งค่าคุณสมบัตินี้เป็น BINARY_TYPE_ARRAYBUFFER

## การใช้ Workerman เป็น WebSocket ไคลเอ็นต์

สามารถใช้ [คลาส AsyncTcpConnection](../async-tcp-connection.md) ร่วมกับ [โปรโตคอล ws](about-ws.md) เพื่อทำให้ Workerman สามารถเป็นไคลเอ็นต์ของ WebSocket และเชื่อมต่อกับเซิร์ฟเวอร์ WebSocket ได้ เพื่อการสื่อสารแบบ full-duplex ที่เป็นแบบ real-time สองทางการทำงานได้ 
