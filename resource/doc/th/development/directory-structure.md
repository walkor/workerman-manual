# โครงสร้างไดเรคทอรี
```
Workerman                      // โค้ดหลักของ workerman
    ├── Connection                 // การเชื่อมต่อแบบ socket ที่เกี่ยวข้อง
    │   ├── ConnectionInterface.php// อินเตอร์เฟซการเชื่อมต่อ socket
    │   ├── TcpConnection.php      // คลาสการเชื่อมต่อ Tcp
    │   ├── AsyncTcpConnection.php // คลาสการเชื่อมต่อ Tcp แบบไม่同เวลา
    │   └── UdpConnection.php      // คลาสการเชื่อมต่อ Udp
    ├── Events                     // ไลบรารีเหตุการณ์เครือข่าย
    │   ├── EventInterface.php     // อินเตอร์เฟซไลบรารีเหตุการณ์เครือข่าย
    │   ├── Event.php              // ไลบรารีเหตุการณ์เครือข่าย Libevent
    │   ├── Ev.php                 // ไลบรารีเหตุการณ์เครือข่าย Libev
    │   ├── Swoole.php             // ไลบรารีเหตุการณ์เครือข่าย Swoole
    │   └── Select.php             // ไลบรารีเหตุการณ์เครือข่าย Select
    ├── Lib                        // ไลบรารีสำหรับใช้งานทั่วไป
    │   ├── Constants.php          // ค่าคงที่ที่ถูกกำหนด
    │   └── Timer.php              // ไทเมอร์
    ├── Protocols                  // การเชื่อมต่อที่เกี่ยวข้องกับโปรโตคอล
    │   ├── ProtocolInterface.php  // คลาสอินเตอร์เฟซโปรโตคอล
    │   ├── Http                   // โปรโตคอล Http ที่เกี่ยวข้อง
    │   │   ├── Chunk.php    // คลาส chunk ของ http
    │   │   ├── Request.php  // คลาสการร้องขอ http
    │   │   ├── Response.php  // คลาสการตอบรับ http
    │   │   ├── ServerSentEvents.php  // คลาส SSE
    │   │   ├── Session
    │   │   │   ├── FileSessionHandler.php  // คลาสการจัดการ session ผ่านไฟล์
    │   │   │   └── RedisSessionHandler.php // คลาสการจัดการ session ผ่าน redis
    │   │   ├── Session.php  // คลาส session
    │   │   └── mime.types   // ไฟล์ map ของ mime
    │   ├── Http.php               // การทำงานของโปรโตคอล Http
    │   ├── Text.php               // การทำงานของโปรโตคอล Text
    │   ├── Frame.php              // การทำงานของโปรโตคอล Frame
    │   └── Websocket.php          // การทำงานของโปรโตคอล websocket
    ├── Worker.php                 // Worker
    ├── WebServer.php              // WebServer
    └── Autoloader.php             // โมดูลการโหลดคลาสอัตโหลด
```
