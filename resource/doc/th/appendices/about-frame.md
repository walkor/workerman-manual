# โปรโตคอล frame
``` (ต้องใช้ Workerman เวอร์ชัน >= 3.3.0) ```

Workerman กำหนดโปรโตคอลที่เรียกว่า frame ซึ่งรูปแบบของโปรโตคอลคือ ``` ความยาวของแพ็คทั้งหมด + ส่วนของแพ็ค``` โดยที่ความยาวของแพ็คเป็นจำนวนเต็ม 4 ไบต์ของระบบเครือข่าย และส่วนของแพ็คอาจเป็นข้อความธรรมดาหรือข้อมูลทวีคูณ

ด้านล่างคือการบรรลุการทำงานของโปรโตคอล frame
```php
class Frame
{

    public static function input($buffer, TcpConnection $connection)
    {
        if(strlen($buffer)<4)
        {
            return 0;
        }
        $unpack_data = unpack('Ntotal_length', $buffer);
        return $unpack_data['total_length'];
    }

    public static function decode($buffer)
    {
        return substr($buffer, 4);
    }

    public static function encode($buffer)
    {
        $total_length = 4 + strlen($buffer);
        return pack('N', $total_length) . $buffer;
    }
}
```
