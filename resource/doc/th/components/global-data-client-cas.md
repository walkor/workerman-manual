# ตัวอย่างการใช้งาน CAS

```php
$global = new GlobalData\Client('127.0.0.1:2207');

// ทำการกำหนดค่าเริ่มต้นให้กับ user_list
$global->user_list = array(1,2,3);

// การเพิ่มค่าเข้าไปใน user_list โดยใช้วิธี CAS
do
{
    $old_value = $new_value = $global->user_list;
    $new_value[] = 4;
}
while(!$global->cas('user_list', $old_value, $new_value));

var_export($global->user_list);
```
ในตัวอย่างนี้ เราใช้ CAS เพื่อเพิ่มค่าเข้าใน user_list โดยถ้ามีการเปลี่ยนแปลงของค่าดังกล่าวในระหว่างที่เราอ่านและเขียนค่าใหม่ จะไม่สามารถทำการเขียนค่าใหม่ได้และจะทำการลองอีกครั้งจนกว่าจะสามารถเขียนค่าใหม่ได้ ซึ่งเป็นวิธีการแก้ปัญหาการเข้าถึงค่าข้อมูลจากหลายๆคนพร้อมกันโดย CAS 
