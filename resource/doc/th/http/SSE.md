SSE 
**คุณสมบัตินี้ต้องใช้ workerman >= 4.0.0**

SSE หรือ Server-sent Events คือเทคโนโลยีการส่งข้อมูลจากเซิร์ฟเวอร์มายังไคลเอนต์ ซึ่งกลไกรพื้นฐานคือเมื่อไคลเอนต์ส่งคำขอ HTTP ที่มีส่วนหัว `Accept: text/event-stream` แล้วเชื่อมต่อจะไม่ปิด ซึ่งเซิร์ฟเวอร์สามารถส่งข้อมูลไปยังไคลเอนต์ได้ตลอดเชื่อมต่อนั้น

ความแตกต่างระหว่าง SSE กับ WebSocket คือ:
* SSE สามารถส่งข้อมูลไปยังไคลเอนต์เท่านั้น ในขณะที่ WebSocket สามารถสื่อสารไปมาได้ทั้งสองทาง
* SSE สนับสนุนการเชื่อมต่อใหม่โดยอัตโนมัติ ในขณะที่ WebSocket ต้องการการดำเนินการเพิ่มเติม
* SSE สามารถส่งข้อมูลข้อความที่เข้าใจได้เป็น UTF-8 เท่านั้น ข้อมูลไบนารีต้องถูกเข้ารหั code เป็น UTF-8 ก่อนการส่ง ในขณะที่ WebSocket สนับสนุนการส่งข้อมูลเป็น UTF-8 และข้อมูลไบนารีโดยปกติ
* SSE ประกอบด้วยประเภทข้อความของตัวเอง ในขณะที่ WebSocket ต้องการการดำเนินการเพิ่มเติม

### ตัวอย่าง
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\ServerSentEvents;
use Workerman\Protocols\Http\Response;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // หากหัวข้อ Accept เป็น text/event-stream แสดงว่าเป็นคำขอ SSE
    if ($request->header('accept') === 'text/event-stream') {
        // ส่งการตอบกลับที่มีหัวข้อ Content-Type: text/event-stream ก่อน
        $connection->send(new Response(200, ['Content-Type' => 'text/event-stream'], "\r\n"));
        // กำหนดการส่งข้อมูลไปยังไคลเอนต์เป็นระยะ ๆ
        $timer_id = Timer::add(2, function () use ($connection, &$timer_id){
            // เมื่อเชื่อมต่อปิดต้องลบตัวจัดการเวลาเพื่อป้องกันการระการเพิ่มเติมที่จะ导致หัวข้อหลุด
            if ($connection->getStatus() !== TcpConnection::STATUS_ESTABLISHED) {
                Timer::del($timer_id);
                return;
            }
            // ส่งอีเวนต์ข้อความพร้อมข้อมูลว่า "hello" และไม่จำเป็นต้องส่งรหัสข้อความ
            $connection->send(new ServerSentEvents(['event' => 'message', 'data' => 'hello', 'id'=>1]));
        });
        return;
    }
    $connection->send('ok');
};

// รันเวิร์กเกอร์
Worker::runAll();
```

รหัส JavaScript ของไคลเอนต์
```js
var source = new EventSource('http://127.0.0.1:8080');
source.addEventListener('message', function (event) {
  var data = event.data;
  console.log(data); // ผลลัพธ์ hello
}, false);
```
