# บริการเซิร์ฟเวอร์ GlobalData component
**``` (คำเรียกใช้ Workerman เวอร์ชัน >= 3.3.0) ```**

# __construct
```php
void \GlobalData\Server::__construct([string $listen_ip = '0.0.0.0', int $listen_port = 2207])
```

สร้างอินสแตนซ์ของเซิร์ฟเวอร์ \GlobalData\Server

### พารามิเตอร์
 ``` listen_ip ```

ที่อยู่ไอพีของเครื่องที่จะฟัง ถ้าไม่ระบุ ค่าเริ่มต้นคือ ```0.0.0.0```

 ``` listen_port ```

พอร์ตที่จะฟัง ถ้าไม่ระบุ ค่าเริ่มต้นคือ 2207

## ตัวอย่าง
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// ฟังก์ชันการฟัง
$worker = new GlobalData\Server('127.0.0.1', 2207);

Worker::runAll();
```
