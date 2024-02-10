# เพิ่ม
**``` (ต้องการ Workerman เวอร์ชั่น >= 3.3.0) ```**

```php
bool \GlobalData\Client::add(string $key, mixed $value)
```

การเพิ่มแบบอะตอมิก หาก key มีอยู่แล้ว จะคืนค่า false

## พารามิเตอร์

 ``` $key ```

คีย์ค่า (เช่น ```$global->abc``` ก็คือคีย์)

 ``` $value ```
ค่าที่จะจัดเก็บ

## ค่าที่คืน
ถ้าสำเร็จจะคืนค่า true ถ้าไม่สำเร็จจะคืนค่า false

## ตัวอย่าง

```php
$global = new GlobalData\Client('127.0.0.1:2207');

if($global->add('some_key', 10))
{
    // กำหนดค่า $global->some_key สำเร็จ
    echo "เพิ่มค่าสำเร็จ " , $global->some_key;
}
else
{
    // $global->some_key มีอยู่แล้ว การกำหนดค่าล้มเหลว
    echo "เพิ่มล้มเหลว " , var_export($global->some_key);
}
```
