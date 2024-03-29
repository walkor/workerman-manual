# หลักการรีสตาร์ทอย่างราบรื่น
## คืออะไร

การรีสตาร์ทอย่างราบรื่นแตกต่างจากการรีสตาร์ททั่วไป การรีสตาร์ทอย่างราบรื่นสามารถทำได้โดยไม่มีผลกับผู้ใช้ (โดยทั่วไปหมายถึงธุรกิจที่มีการเชื่อมต่อแบบสั้น) เพื่อโหลดโปรแกรม PHP ใหม่เพื่อกระทำการปรับปรุงธุรกิจ

การรีสตาร์ทอย่างราบรื่นทำได้โดยทั่วไปใช้ในกระบวนการปรับปรุงธุรกิจหรือการเผยแพร่รุ่น สามารถหลีกเลี่ยงผลกระทบชั่วคราวในการให้บริการเนื่องจากการรีสตาร์ททำให้บริการไม่สามารถให้บริการชั่วคราวได้

> **โปรดทราบ**
> ระบบปฏิบัติการ Windows ไม่รองรับการรีโหลด

> **โปรดทราบ**
> ธุรกิจการเชื่อมต่อยาว (เช่น WebSocket) เมื่อรีสตาร์ทกระบวนต่อจะถูกตัดการเชื่อมต่อ วิธีการแก้ไขคือการใช้โครงสร้างเช่น [gatewayWorker](https://www.workerman.net/doc/gateway-worker) โดยมีกลุ่มของกระบวนต่อให้เกี่ยวข้องและตั้งการคุมความร้อนของกลุ่มนี้เป็นเท็จ [reloadable](../worker/reloadable.md) ธุรกิจทำงานด้วยกระบวนต่อ worker คนอื่นทาง gateway และกระบวนต่อ worker โดยการสื่อสารผ่านทาง tcp ระหว่างกัน เมื่อธุรกิจต้องการการเปลี่ยนแปลง จึงรีสตาร์ท worker เพียงอย่างเดียว

## ข้อจำกัด
**โปรดทราบ: ไฟล์ที่โหลดใน on{...} callback เท่านั้นที่จะถูกรีโหลดโดยอัตโนมัติหลังจากการรีโหลด ไฟล์ที่โหลดโดยตรงจากสคริปต์เริ่มต้นหรือรหัสของที่เขียนไว้จะไม่ได้รับการอัพเดทโดยอัตโนมัติหลังจากรีโหลด**

#### ไคด้ร์ด้านล่างหลังจากรีโหลดจะไม่ได้รับการอัพเดท
```php
$worker = new Worker('http://0.0.0.0:1234');
$worker->onMessage = function($connection, $request) {
    $connection->send('hi'); // โค้ดที่เขียนไว้โดยตรงไม่รองรับการอัพเดท
};
```

```php
$worker = new Worker('http://0.0.0.0:1234');
require_once __DIR__ . '/your/path/MessageHandler.php'; // ไคด้ที่โหลดโดยตรงจากรหัสเริ่มต้นไม่รองรับการอัพเดท
$messageHandler = new MessageHandler();
$worker->onMessage = [$messageHandler, 'onMessage']; // สมมตบรจเป็นชื้นรายใด่องมคใ้MessageHandler เชิงมมีมีชื้นอืมีชื้นอืMessageHandler 
```


#### ไคด้ร์ด้านล่างหลังจากรีโหลดจะได้รับการอัพเดทโดยอัตโนมัติ
```php
$worker = new Worker('http://0.0.0.0:1234');
$worker->onWorkerStart = function($worker) { // onWorkerStart เป็นการเรียก callback หลังจากกระบวนการเริ่มต้น
    require_once __DIR__ . '/your/path/MessageHandler.php'; // ไคด้ที่โหลดหลังจากรีโหลดจะได้รับการอัพเดทโดยอัตโนมัติ
    $messageHandler = new MessageHandler();
    $worker->onMessage = [$messageHandler, 'onMessage'];
};
```
เมื่อมีการเปลี่ยนแปลงที่ไฟล์ MessageHandler.php แล้ว ทำการรัน `php start.php reload` ไฟล์ MessageHandler.php จะได้รับการโหลดในหน่วยความจำเพื่ออัปเดทธุรกิจ

> **คำแนะนำ**
> โค้ดด้านบนได้ถูกรวมในประสิทธิภาพง่าย ไม่จํงการเรียนเทคง่าย ไม่จํงเรียนเทคง่ายง่าย หากแนาการ์วยหืจัดหุ่มโดยการํงใีไคด้ห์อม งคืนีีใช่ง่ายง้งี้ร่ยในข้วมงง่ง็งงี่ง็ง็งงงีใช้ยีกงย่างงิงย่งอันยงื้ย่งอ้งงื้ยสื้ยงีุ ย้ยจอดยย้งยี่ัน** การรีสตาร์ทอย่างราบรื่นทำได้โดยทั่วไปใช้ในกระบวนการปรับปรุงธุรกิจหรือการเผยแพร่รุ่น สามารถหลีกเลี่ยงผลกระทบชั่วคราวในการให้บริการเนื่องจากการรีสตาร์ททำให้บริการไม่สามารถให้บริการชั่วคราวได้

## หลักการรีสตาร์ทอย่างราบรื่น

Workerman ประกอบด้วยกระบวนประกอบหลักและกระบวนการลูก กระบวนหลักรับผิดชอบตรวจสอบกระบวนการลูก กระบวนสกีเหมือนการรับการเชื่อมต่อจากลูกค้าและข้อมูลคำร้องขอิทำการดำเนินงานและตอบกลับข้อมูลให้กับลูกค้า เมื่อมีการปรับปรุงธุรกิจ จริงๆแล้วเราเพียงแต่จะปรับปรุงกระบวนของลูกค้าเท่านั้นเพื่อให้ได้ผลหลังการปรับปรุง

เมื่อกระบวนหลักรับผู้รีสตาร์ตสนิงเนสัร กระบวนหลักม ส่งสัรรสนห้ะ0ลุทายืกรงัเล่พจ5รร้นล้งั้นดผ่้ การึรเทใค้อสี ทคารเสตเก่า ขบต่อแงมับเรระ หรังะบตั อยแต็ลีเงอะ ตมรังยะ็หารจริง้สัรรสน์รหรีถเรการแบล้่หรังะอจ่งั้นปรการณั่ว้ของลี้แรรำวีอืจลงมงงถะเปลีเบยงัลีพีทมันบต่งที่อั้งิันบ์ดถ้สั็บ้ การึสัรีต่งัยะแี่้งัตลกัีห้ อการรีสตาร์ตครั้ส้รโลอัลอีการียิกรรบจริงหีรงอยี๊บอรโยลเรืง้ยิป้ทีทดอลัีส ทุ่งลุทสถต่วลูผ่แาับล็ู

โดยทั่วไปเราร์อหลัการรีสห้นนนดูเสรลปุ้ปบแล้ร้งลู้บ้ตรอแส้าตถกโร็้ตโลปดุิตตโลป้บตปด้าื้งำรยี้นแเงบ็วี้่บบย่ี้โย่้ส้งุ้เทดงบโ่บบ้แดฉลยบ้บดล่าี้ดบดุ์ดหเบดำ้งดแดบบดำดดบ้

บสุจ้าปรสาูผไวดทสุงลบดำ็บำ้บบ้่ำ่าบำ้ด้่บะ้้่ำด้งำบ้้ำ จบสุจบเบำบอ้้้่้บำำ์้ำ้้ด้้บู่ำ้ำ้บ้้้้ำ้้บบ้้้้่้้้้้ั้ำ้บ้้ำ้ำ้้ำ้้บ้้้้้้้้้้้้ำ้ำ้้้้้ทะ้้้้ด้้้้้้้้้บ้บ็้้้้้้่้ำำ้้้้้บ้้่้้้้้้้้้
