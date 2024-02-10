# ไอดี
ต้องการ ``` (workerman >= 3.2.1) ```

## คำอธิบาย:
```php
int Worker::$id
```

เลขที่บอกถึง id ของ worker ปัจจุบัน อยู่ในช่วงระหว่าง ```0``` ถึง ```$worker->count-1```.

สมบัตินี้มีประโยชน์มากสำหรับการแยก worker process ตัวอย่างเช่น หากมีกรณีที่มี worker คนนึงมีหลายๆ process และนักพัฒนาต้องการตั้งค่าตัวจับเวลาไว้ใน process คนเดียวเท่านั้น ก็สามารถทำได้โดยการระบุ id ของ process ได้ เช่น ต้องการตั้งค่าตัวจับเวลาอยู่ใน process ที่มี id เป็น 0 เท่านั้น (ดูตัวอย่าง)

## หมายเหตุ:

หลังจากที่โปรเซสถูกเริ่มต้นใหม่ ค่า id จะไม่เปลี่ยนไป

การจัดสรร id ของ process จะเป็นอิงกับแต่ละ worker instance แต่ละ worker instance จะเริ่มต้นการจัดสรร id ของ process ของตัวเองที่ 0 ดังนั้น id ของ process ของ worker instance มีโอกาสที่จะซ้ำกัน แต่ id ของ process ของ worker instance คนละตัวจะไม่ซ้ำกัน ตัวอย่างเช่น:

```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// worker instance ที่ 1 มี 4 โปรเซส โปรเซส id จะเป็น 0, 1, 2, 3 ตามลำดับ
$worker1 = new Worker('tcp://0.0.0.0:8585');
// กำหนดให้เริ่มต้น โปรเซส 4 ตัว
$worker1->count = 4;
// เมื่อโปรเซสเริ่มต้นแล้วให้พิมพ์ id ของโปรเซสปัจจุบันไว้ที่ $worker1->id
$worker1->onWorkerStart = function($worker1)
{
    echo "worker1->id={$worker1->id}\n";
};

// worker instance ที่ 2 มี 2 โปรเซส โปรเซส id จะเป็น 0, 1 ตามลำดับ
$worker2 = new Worker('tcp://0.0.0.0:8686');
// กำหนดให้เริ่มต้น โปรเซส 2 ตัว
$worker2->count = 2;
// เมื่อโปรเซสเริ่มต้นแล้วให้พิมพ์ id ของโปรเซสปัจจุบันไว้ที่ $worker2->id
$worker2->onWorkerStart = function($worker2)
{
    echo "worker2->id={$worker2->id}\n";
};

// เริ่มการทำงาน worker
Worker::runAll();
```
ผลลัพธ์จะเป็นดังนี้
```php
worker1->id=0
worker1->id=1
worker1->id=2
worker1->id=3
worker2->id=0
worker2->id=1
```
หมายเหตุ: ระบบปฏิบัติการ windows เนื่องจากไม่รองรับการตั้งค่าจำนวนโปรเซส count จึง id มีเพียงแค่หนึ่งตัวที่ 0 เท่านั้น ระบบปฏิบัติการ windows ไม่รองรับการรัน worker 2 instance จากไฟล์เดียวกัน ดังนั้นตัวอย่างนี้ไม่สามารถรันได้ในระบบปฏิบัติการ windows.

## ตัวอย่าง
worker อย่างเดียวมี 4 โปรเซส ทำการตั้งค่าตัวจับเวลาไว้เฉพาะในโปรเซสที่มี id เป็น 0.

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8585');
$worker->count = 4;
$worker->onWorkerStart = function($worker)
{
    // ตั้งค่าตัวจับเวลาไว้เฉพาะในโปรเซสที่มี id เป็น 0 โดยไม่ต้องไปก่อนตัวจับเวลาให้กับ process ที่มี id เป็น 1, 2, 3
    if($worker->id === 0)
    {
        Timer::add(1, function(){
            echo "4 โปรเซส worker ตั้งตัวจับเวลาไว้เฉพาะในโปรเซสที่มี id เป็น 0\n";
        });
    }
};
// รัน worker ทั้งหมด
Worker::runAll();
```
