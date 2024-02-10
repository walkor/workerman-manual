# ตัวอย่างบางส่วน

## ตัวอย่างที่ 1

### การกำหนดโปรโตคอล
  * ส่วนหัวคงที่ 10 ไบต์ ที่ใช้สำหรับเก็บความยาวของแพ็คเกจทั้งหมด หากจำนวนไม่พอ ให้เติม 0
  * รูปแบบข้อมูลเป็น xml

### ตัวอย่างแพ็คเกจข้อมูล
```xml
0000000121<?xml version="1.0" encoding="ISO-8859-1"?>
<request>
    <module>user</module>
    <action>getInfo</action>
</request>
```
เครื่องหมาย 0000000121 แทนความยาวของแพ็คเกจทั้งหมด ตามด้วยเนื้อหาของแพ็คเกจรูปแบบ xml

### การปฏิบัติตามโปรโตคอล
```php
namespace Protocols;
class XmlProtocol
{
    public static function input($recv_buffer)
    {
        if(strlen($recv_buffer) < 10)
        {
            // ความยาวไม่พอ 10 บายต์ คืนค่า 0 เพื่อรอข้อมูลต่อไป
            return 0;
        }
        // คืนค่าความยาวของแพ็คเกจทั้งหมด รวมถึงความยาวของหัวและเนื้อหา
        $total_len = base_convert(substr($recv_buffer, 0, 10), 10, 10);
        return $total_len;
    }

    public static function decode($recv_buffer)
    {
        // เนื้อหาแพ็คเกจ
        $body = substr($recv_buffer, 10);
        return simplexml_load_string($body);
    }

    public static function encode($xml_string)
    {
        // ความยาวของเนื้อหาและหัวแพ็คเกจ
        $total_length = strlen($xml_string)+10;
        // เติม 0 ให้ครบ 10 บายต์ หากจำนวนไม่พอ
        $total_length_str = str_pad($total_length, 10, '0', STR_PAD_LEFT);
        // คืนค่าข้อมูล
        return $total_length_str . $xml_string;
    }
}
```

## ตัวอย่างที่ 2

### การกำหนดโปรโตคอล
  * ส่วนหัว 4 ไบต์ เป็น unsigned int ที่เรียงตามระบบเครือข่าย เพื่อระบุความยาวของแพ็คเกจทั้งหมด
  * ข้อมูลเป็นสตริง Json

### ตัวอย่างแพ็คเกจข้อมูล
<pre>
****{"type":"message","content":"hello all"}
</pre>

เครื่องหมาย * 4 หมายถึง unsigned int ตามระบบเครือข่าย ไม่สามารถมองเห็นได้ ตามด้วยข้อมูลแบบ Json ของเนื้อหาแพ็คเกจ

### การปฏิบัติตามโปรโตคอล
```php
namespace Protocols;
class JsonInt
{
    public static function input($recv_buffer)
    {
        // ข้อมูลที่ได้รับยังไม่ครบ 4 ไบต์ ไม่สามารถระบุความยาวของแพ็คเกจได้ คืนค่า 0 เพื่อรอข้อมูลต่อไป
        if(strlen($recv_buffer)<4)
        {
            return 0;
        }
        // ใช้ฟังก์ชั่น unpack เพื่อแปลง 4 ไบต์แรกเป็นตัวเลข ซึ่งคือความยาวของแพ็คเกจทั้งหมด
        $unpack_data = unpack('Ntotal_length', $recv_buffer);
        return $unpack_data['total_length'];
    }

    public static function decode($recv_buffer)
    {
        // ลบส่วนหัว 4 ไบต์ เพื่อให้เหลือเฉพาะเนื้อหา Json ของแพ็คเกจ
        $body_json_str = substr($recv_buffer, 4);
        // ถอดรหัส Json
        return json_decode($body_json_str, true);
    }

    public static function encode($data)
    {
        // การเข้ารหัส Json เพื่อให้ได้เป็นเนื้อหาแพ็คเกจ
        $body_json_str = json_encode($data);
        // คำนวณความยาวของแพ็คเกจทั้งหมด รวมถึง 4 ไบต์แรก และจำนวนไบต์ของเนื้อหา
        $total_length = 4 + strlen($body_json_str);
        // คืนค่าข้อมูลที่เข้ารหัสแล้ว
        return pack('N',$total_length) . $body_json_str;
    }
}
```
## ตัวอย่างที่สาม (การอัปโหลดไฟล์โดยใช้โปรโตคอลไบนารี)
### คำนิยามของโปรโตคอล
```C
struct
{
  unsigned int total_len;  // ความยาวของแพ็กทั้งหมดในรูปแบบของตัวเลขแบบบิ๊กเอ็น การจัดลำดับข้อมูลผ่านเครือข่าย
  char         name_len;   // ความยาวของชื่อไฟล์
  char         name[name_len]; // ชื่อไฟล์
  char         file[total_len - BinaryTransfer::PACKAGE_HEAD_LEN - name_len]; // ข้อมูลไฟล์
}
```
### ตัวอย่างของโปรโตคอล

<pre> *****logo.png****************** </pre>

ที่อยู่บนส่วนหัวของสี่ตัวอักษร * แทนข้อมูลตัวเลขแบบบิ๊กเอ็นที่เป็นอักขระที่ไม่สามารถมองเห็นได้ ตัวอักษรที่ 5 ถูกใช้เพื่อเก็บความยาวของชื่อไฟล์ ตามโดยต้นจะเป็นข้อมูลของไฟล์แบบไบนารี

### การปรับใช้โปรโตคอล
```php
namespace Protocols;
class BinaryTransfer
{
    // ความยาวของส่วนหัวของโปรโตคอล
    const PACKAGE_HEAD_LEN = 5;

    public static function input($recv_buffer)
    {
        // หากข้อมูลไม่เพียงพอสำหรับความยาวของส่วนหัวของโปรโตคอล ให้รอโปรเซสต่อ
        if(strlen($recv_buffer) < self::PACKAGE_HEAD_LEN)
        {
            return 0;
        }
        // แกะการจัดลำดับข้อมูล
        $package_data = unpack('Ntotal_len/Cname_len', $recv_buffer);
        // คืนค่าความยาวของแพ็ก
        return $package_data['total_len'];
    }


    public static function decode($recv_buffer)
    {
        // แกะการจัดลำดับข้อมูล
        $package_data = unpack('Ntotal_len/Cname_len', $recv_buffer);
        // ความยาวของชื่อไฟล์
        $name_len = $package_data['name_len'];
        // ตัดชื่อไฟล์ออกจากข้อมูลที่ได้
        $file_name = substr($recv_buffer, self::PACKAGE_HEAD_LEN, $name_len);
        // ตัดข้อมูลไฟล์แบบไบนารีออกจากข้อมูลที่ได้
        $file_data = substr($recv_buffer, self::PACKAGE_HEAD_LEN + $name_len);
         return array(
             'file_name' => $file_name,
             'file_data' => $file_data,
         );
    }

    public static function encode($data)
    {
        // สามารถเข้ารหัสข้อมูลที่ต้องการส่งไปยังไคลเอ็นต์ตามความต้องการของตนเองได้ ในที่นี้มีแค่เพียงส่งข้อมูลเป็นข้อความต้นฉบับ
        return $data;
    }
}
```

### ตัวอย่างการใช้โปรโตคอลที่เซิร์ฟเวอร์

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('BinaryTransfer://0.0.0.0:8333');
// บันทึกไฟล์ไว้ที่ tmp
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $save_path = '/tmp/'.$data['file_name'];
    file_put_contents($save_path, $data['file_data']);
    $connection->send("upload success. save path $save_path");
};

Worker::runAll();
```

### ตัวอย่างการใช้ไคลเอ็นต์ของโปรแกรม client.php (ตรงนี้ใช้ php จำลองเป็นไคลเอ็นต์สำหรับการอัปโหลด)
```php
<?php
/** ไคลเอ็นต์อัปโหลดไฟล์ **/
// ที่อยู่สำหรับอัปโหลด
$address = "127.0.0.1:8333";
// ตรวจสอบพารามิเตอร์เส้นทางของไฟล์ที่จะอัปโหลด
if(!isset($argv[1]))
{
   exit("use php client.php \$file_path\n");
}
// พารามิเตอร์เส้นทางไฟล์ที่จะอัปโหลด
$file_to_transfer = trim($argv[1]);
// ไฟล์ที่จะอัปโหลดไม่มีอยู่ในเครื่อง
if(!is_file($file_to_transfer))
{
    exit("$file_to_transfer not exist\n");
}
// สร้างการเชื่อมต่อแบบซ็อกเก็ต
$client = stream_socket_client($address, $errno, $errmsg);
if(!$client)
{
    exit("$errmsg\n");
}
// กำหนดให้เป็นการบล็อค
stream_set_blocking($client, 1);
// ชื่อไฟล์
$file_name = basename($file_to_transfer);
// ความยาวของชื่อไฟล์
$name_len = strlen($file_name);
// ข้อมูลไฟล์แบบไบนารี
$file_data = file_get_contents($file_to_transfer);
// ความยาวของส่วนหัวของโปรโตคอล 4 บายต์ของชุดข้อมูล + 1 บายต์ของความยาวของชื่อไฟล์
$PACKAGE_HEAD_LEN = 5;
// พร็อตโตคอลแพ็ก
$package = pack('NC', $PACKAGE_HEAD_LEN  + strlen($file_name) + strlen($file_data), $name_len) . $file_name . $file_data;
// ดำเนินการอัปโหลด
fwrite($client, $package);
// ปริ้นผลลัพธ์
echo fread($client, 8192),"\n";
```

### ตัวอย่างการใช้ไคลเอ็นต์
รันคำสั่งในหน้าโปรแกรมคอมพิวเตอร์ ```php client.php <เส้นทางของไฟล์>```

ยกตัวอย่าง ```php client.php abc.jpg```
## ตัวอย่างที่สี่ (การอัปโหลดไฟล์โดยใช้โปรโตคอลข้อความ)

### กำหนดโปรโตคอล

json+ขึ้นบรรทัดใหม่ ซึ่งมีข้อมูลอยู่ใน json ประกอบด้วยชื่อไฟล์และข้อมูลไฟล์ที่ถูก encode ด้วย base64 (ขนาดข้อมูลจะเพิ่มขึ้น 1/3)

### ตัวอย่างโปรโตคอล

{"file_name":"logo.png","file_data":"PD9waHAKLyo......"}\n

โปรดทราบว่าท้ายข้อความมีเครื่องหมายยังหยังเพียงอย่างเดียว ใน PHP มีสัญลักษณ์คำของเครื่องหมายคำพูดคู่ ```"\n"```

### การปฏิบัติตามโปรโตคอล

```php
namespace Protocols;
class TextTransfer
{
    public static function input($recv_buffer)
    {
        $recv_len = strlen($recv_buffer);
        if($recv_buffer[$recv_len-1] !== "\n")
        {
            return 0;
        }
        return strlen($recv_buffer);
    }

    public static function decode($recv_buffer)
    {
        // ถอดรหัส
        $package_data = json_decode(trim($recv_buffer), true);
        // ดึงชื่อไฟล์ออกมา
        $file_name = $package_data['file_name'];
        // ดึงข้อมูลไฟล์ที่ถูก encode ด้วย base64
        $file_data = $package_data['file_data'];
        // base64_decode เพื่อกลับไปเป็นข้อมูลไฟล์แบบ binary ต้นฉบับ
        $file_data = base64_decode($file_data);
        // ส่งข้อมูลกลับ
        return array(
             'file_name' => $file_name,
             'file_data' => $file_data,
         );
    }

    public static function encode($data)
    {
        // สามารถ encode ข้อมูลที่จะส่งกลับให้กับลูกค้าตามต้องการของตัวเองได้ ที่นี่จะให้เป็นข้อความเหมือนเดิม
        return $data;
    }
}

```

### ตัวอย่างการใช้โปรโตคอลที่เซิร์ฟเวอร์
คำอธิบาย: รูปแบบการเขียนเหมือนกับกรณีการอัปโหลดไฟล์ในรูปแบบ ไบนารี ซึ่งจะสามารถเปลี่ยนโปรโตคอลได้โดยไม่ต้องเปลี่ยนแปลงโค้ดธุรกิจใด ๆ

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('TextTransfer://0.0.0.0:8333');
// บันทึกไฟล์ไว้ที่ tmp
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $save_path = '/tmp/'.$data['file_name'];
    file_put_contents($save_path, $data['file_data']);
    $connection->send("upload success. save path $save_path");
};

Worker::runAll();
```

### ตัวอย่างการใช้โปรโตคอลที่เซิร์ฟเวอร์
คำอธิบาย: รูปแบบการเขียนเหมือนกับกรณีการอัปโหลดไฟล์ในรูปแบบ ไบนารี ซึ่งจะสามารถเปลี่ยนโปรโตคอลได้โดยไม่ต้องเปลี่ยนแปลงโค้ดธุรกิจใด ๆ

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('TextTransfer://0.0.0.0:8333');
// บันทึกไฟล์ไว้ที่ tmp
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $save_path = '/tmp/'.$data['file_name'];
    file_put_contents($save_path, $data['file_data']);
    $connection->send("upload success. save path $save_path");
};

Worker::runAll();
```
