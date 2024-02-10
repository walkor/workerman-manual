# โปรโตคอลข้อความ
> Workerman ได้กำหนดโปรโตคอลข้อความที่เรียกว่า "text" ซึ่งรูปแบบของโปรโตคอลคือ ```แพ็คเกจข้อมูล+เครื่องหมายขึ้นบรรทัดใหม่``` นั่นคือการเพิ่มเครื่องหมายขึ้นบรรทัดใหม่ที่ด้านท้ายของแต่ละแพ็คเกจเพื่อแสดงถึงปิดแพ็คเกจ

ตัวอย่างของ buffer1 และ buffer2 ด้านล่างนี้เป็นตัวอย่างของข้อความที่เข้ากันกับโปรโตคอล text:

```php
// ข้อความ + เปลี่ยนบรรทัด
$buffer1 = 'abcdefghijklmn
';
// ใน php \n ภายใน double quotes แทนด้วยเครื่องหมายขึ้นบรรทัดใหม่ เช่น "\n"
$buffer2 = '{"type":"say", "content":"hello"}'."\n";

// เชื่อมต่อกับเซิร์ฟเวอร์ผ่าน socket
$client = stream_socket_client('tcp://127.0.0.1:5678');
// ส่งข้อมูล buffer1 โดยใช้โปรโตคอล text
fwrite($client, $buffer1);
// ส่งข้อมูล buffer2 โดยใช้โปรโตคอล text
fwrite($client, $buffer2);
```

โปรโตคอล text ง่ายต่อการใช้งาน หากนักพัฒนาต้องการโปรโตคอลที่เป็นของตัวเอง เช่น สำหรับการสื่อสารข้อมูลกับแอพพลิเคชันมือถือหรือการสื่อสารกับฮาร์ดแวร์ สามารถพิจารณาใช้โปรโตคอล text เนื่องจากการพัฒนาและการDebug มีความสะดวกสบายมาก

**การDebug โปรโตคอล text**

> โปรโตคอล text สามารถใช้ telnet client เพื่อ Debug ตัวอย่างเช่นด้านล่างนี้:

สร้างไฟล์ test.php

```php
require_once __DIR__ . '/Workerman/Autoloader.php';
use Workerman\Worker;

$text_worker = new Worker("text://0.0.0.0:5678");

$text_worker->onMessage =  function($connection, $data)
{
    var_dump($data);
    $connection->send("hello world");
};

Worker::runAll();
```

ทำการ ```php test.php start``` และจะแสดงผลดังนี้

```bash
php test.php start
Workerman[test.php] start in DEBUG mode
----------------------- WORKERMAN -----------------------------
Workerman version:3.2.7          PHP version:5.4.37
------------------------ WORKERS -------------------------------
user          worker        listen                         processes status
root          none          myTextProtocol://0.0.0.0:5678   1         [OK]
----------------------------------------------------------------
Press Ctrl-C to quit. Start success.
```

เปิด terminal ใหม่ แล้วใช้ telnet ทดสอบ (แนะนำให้ใช้ telnet ของระบบปฏิบัติการ Linux)

ถ้าทดสอบบนเครื่อง local
ให้เปิด terminal และใช้คำสั่ง telnet 127.0.0.1 5678
จากนั้นใส่ hi แล้วกด enter
จะได้รับข้อมูล hello world\n
```bash
telnet 127.0.0.1 5678
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
hi
hello world
```
