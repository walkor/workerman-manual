# การเกิดข้อผิดพลาด (onError)
## คำอธิบาย:
```PHP
callback Worker::$onError
```

เมื่อเกิดข้อผิดพลาดในการเชื่อมต่อของไคลเอ็นต์ และปล่อยให้เกิดผลเนื่องจากทำงาน


ขณะนี้มีประเภทข้อผิดพลาดดังนี้

1. เรียกใช้ Connection::send ทำให้การส่งข้อมูลล้มเหลวเนื่องจากการเชื่อมต่อของไคลเอ็นต์ตัดการเชื่อมต่อ (จะทำให้เกิดการเรียกใช้ onCLose ต่อมา) ```(code:WORKERMAN_SEND_FAIL msg:client closed)```

2. หลังจากเกิด onBufferFull (บัฟเฟอร์ส่งเต็ม) แล้ว การเรียกใช้ Connection::send อีกครั้ง และบัฟเฟอร์การส่งยังคงเต็มสถานะทำให้การส่งล้มเหลว (ไม่ทำให้เกิดการเรียกใช้ onClose)```(code:WORKERMAN_SEND_FAIL msg:send buffer full and drop package)```

3. เมื่อการเชื่อมต่อแบบ AsyncTcpConnection เชื่อมต่อล้มเหลว (จะทำให้เกิดการเรียกใช้ onClose ต่อมา)```(code:WORKERMAN_CONNECT_FAIL msg:stream_socket_client returns error message)```



## พารามิเตอร์ของฟังก์ชัน callback

 ``` $connection ```

อ็อบเจ็กต์เชื่อมต่อ หรือ [TcpConnection instance](../tcp-connection.md) ที่ใช้ในการดำเนินการกับการเชื่อมต่อของไคลเอ็นต์ เช่น [ส่งข้อมูล](../tcp-connection/send.md) [ปิดการเชื่อมต่อ](../tcp-connection/close.md) ฯลฯ

 ``` $code ```

รหัสข้อผิดพลาด

 ``` $msg ```

ข้อความข้อผิดพลาด


## ตัวอย่าง

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onError = function(TcpConnection $connection, $code, $msg)
{
    echo "error $code $msg\n";
};
// รันเวิร์กเกอร์
Worker::runAll();
```

คำเตือน: นอกเหนือจากการใช้ฟังก์ชันอนิเมะเป็นการตอบรับ ยังสามารถ [ดูตัวอย่างที่นี่](../faq/callback_methods.md) เพื่อใช้วิธีการเขียน callback แบบอื่น ๆ ได้
