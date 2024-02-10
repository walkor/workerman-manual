# คลาสคอนสตรักเตอร์ของคอมโพเนนต์เซิร์ฟเวอร์แชนแนล

**``` (ต้องการ Workerman เวอร์ชัน >= 3.3.0) ```**

# __construct
```php
void \Channel\Server::__construct([string $listen_ip = '0.0.0.0', int $listen_port = 2206])
```

สร้างอ็อบเจ็กต์ของเซิร์ฟเวอร์ \Channel\Server

### พารามิเตอร์
 ``` listen_ip ```

ไอพีแอดเดรสของเครื่องที่ฟัง หากไม่ระบุ ค่าเริ่มต้นคือ```0.0.0.0```

 ``` listen_port ```

พอร์ตที่ฟัง หากไม่ระบุ ค่าเริ่มต้นคือ 2206

## ตัวอย่าง

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// ไม่ระบุพารามิเตอร์จะเป็นการฟังที่ 0.0.0.0:2206 โดยค่าเริ่มต้น
$channel_server = new Channel\Server();

if(!defined('GLOBAL_START'))
{
    Worker::runAll();
}
```
