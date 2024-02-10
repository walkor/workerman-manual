# คอมโพเนนต์แบ่งปันตัวแปร GlobalData
**``` (ต้องใช้เวอร์ชัน Workerman >= 3.3.0) ```**

ที่อยู่ของโค้ด: https://github.com/walkor/GlobalData

## ข้อควรระวัง
GlobalData ต้องการเวอร์ชันของ Workerman ที่มีค่า >= 3.3.0

## การติดตั้ง

`composer require workerman/globaldata`

## หลักการทำงาน

การใช้งานของ GlobalData นี้ใช้เมทอดมายเจิก `__set __get __isset __unset` ของ PHP ในการกระตุ้นการสื่อสารกับเซิร์ฟเวอร์ GlobalData โดยตัวแปรจริงถูกเก็บไว้ที่เซิร์ฟเวอร์ GlobalData ตัวอย่างเช่นเมื่อมีการตั้งค่าคุณสมบัติที่ไม่มีอยู่ในคลาสของไคลเอนต์ จะกระตุ้น `__set` เมทอด ต่อไปนั้นคลาสของไคลเอนต์ในเมทอด `__set` จะส่งคำขอไปยังเซิร์ฟเวอร์ GlobalData เพื่อเก็บตัวแปรหนึ่ง และเมื่อเข้าถึงตัวแปรที่ไม่มีอยู่ในคลาสของไคลเอนต์ จะกระตุ้น `__get` เมทอด ไคลเอนต์จะส่งคำขอไปยังเซิร์ฟเวอร์ GlobalData เพื่ออ่านค่านั้น ๆ ซึ่งจะทำให้การแบ่งปันตัวแปรระหว่างโปรเซสสิร์ด้วยกันเสร็จสิ้น

```php
require_once __DIR__ . '/vendor/autoload.php';

// เชื่อมต่อกับเซิร์ฟเวอร์ Global Data
$global = new GlobalData\Client('127.0.0.1:2207');

// กระตุ้น $global->__isset('somedata') เพื่อตรวจสอบว่าเซิร์ฟเวอร์ได้เก็บคีย์ somedata ไว้หรือยัง
isset($global->somedata);

// กระตุ้น $global->__set('somedata',array(1,2,3)) เพื่อแจ้งให้เซิร์ฟเวอร์เก็บค่าที่เกี่ยวข้องกับsomedata เป็น array(1,2,3)
$global->somedata = array(1,2,3);

// กระตุ้น $global->__get('somedata') เพื่ออ่านค่าที่เก็บเกี่ยวกับsomedata จากเซิร์ฟเวอร์
var_export($global->somedata);

// กระตุ้น $global->__unset('somedata') เพื่อแจ้งเซิร์ฟเวอร์ลบคีย์somedata และค่าที่เก็บเกี่ยวข้อง
unset($global->somedata);
```
