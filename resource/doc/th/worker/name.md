# ชื่อ

## คำอธิบาย:
```php
string Worker::$name
```

กำหนดชื่อของการทำงานปัจจุบันของ Worker instance เพื่อช่วยในการระบุกระบวนการเมื่อใช้คำสั่ง status โดยค่าเริ่มต้นคือ none

## ตัวอย่าง

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// กำหนดชื่อของ instance
$worker->name = 'MyWebsocketWorker';
$worker->onWorkerStart = function($worker)
{
    echo "การเริ่มต้นการทำงานของ Worker...\n";
};
// เรียกใช้ worker
Worker::runAll();
```
