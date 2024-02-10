# workerman/crontab

# คำอธิบาย
`workerman/crontab` เป็นโปรแกรมงานที่ตั้งเวลาทำงานขึ้นบน workerman ที่คล้ายกับ crontab ของ Linux โดย `workerman/crontab` สนับสนุนการตั้งเวลาทำงานทุกวินาที

> การใช้ `workerman/crontab` จำเป็นต้องตั้งค่าโซนเวลาของ PHP ให้ถูกต้อง ไม่งั้นผลการทำงานอาจไม่ตรงกับที่คาดหวัง

## คำอธิบายเวลา
```php
0   1   2   3   4   5
|   |   |   |   |   |
|   |   |   |   |   +------ วันในสัปดาห์ (0 - 6) (วันอาทิตย์=0)
|   |   |   |   +------ เดือน (1 - 12)
|   |   |   +-------- วันในเดือน (1 - 31)
|   |   +---------- ชั่วโมง (0 - 23)
|   +------------ นาที (0 - 59)
+-------------- วินาที (0-59)[สามารถข้ามได้, หากไม่มี 0 ขึ้นตอนที่น้อยที่สุดคือนาที]
```

# การติดตั้ง
```composer require workerman/crontab```

# ตัวอย่าง
```php
<?php
use Workerman\Worker;
require __DIR__ . '/vendor/autoload.php';

use Workerman\Crontab\Crontab;
$worker = new Worker();

// ตั้งค่าโซนเวลาเพื่อหลีกเลี่ยงผลลัพธ์ไม่ตรงกับที่คาดหวัง
date_default_timezone_set('PRC');

$worker->onWorkerStart = function () {
    // ทำงานทุกนาทีที่ 1 วินาที
    new Crontab('1 * * * * *', function(){
        echo date('Y-m-d H:i:s')."\n";
    });
    // ทำงานทุกวัน เวลา 7 โมง 50 นาที โปรดสังเกตว่าข้ามช่องวินาที
    new Crontab('50 7 * * *', function(){
        echo date('Y-m-d H:i:s')."\n";
    });
};

Worker::runAll();
```

> หมายเหตุ: งานที่ตั้งเวลาจะไม่ทำงานทันที ทุกงานที่ตั้งเวลาจะทำงานในนาทีถัดไป

# อินเตอร์เฟซ
**Crontab::destroy()**

ทำลายตัวตั้งเวลา
```php
$crontab = new Crontab('1 * * * * *', function(){
    echo date('Y-m-d H:i:s')."\n";
});
$crontab->destroy();
```
