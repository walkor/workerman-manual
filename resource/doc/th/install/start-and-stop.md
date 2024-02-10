# เริ่มต้นและหยุด

โปรดทราบว่าคำสั่งเริ่มต้นและหยุดของ Workerman จะทำผ่าน command line เท่านั้น

เมื่อต้องการเริ่ม Workerman จะต้องมีไฟล์เริ่มต้น ซึ่งจะกำหนดพอร์ตและโปรโตคอลที่ต้องการที่จะได้รับข้อมูล สามารถอ้างอิงได้ที่ [Workerman Chat](https://www.workerman.net/workerman-chat) เช่นเข้าไปที่ไฟล์ start.php
### เริ่มต้น

เริ่มต้นด้วยการ debug mode

 ```php start.php start```

เริ่มด้วย daemon mode

 ```php start.php start -d```

### หยุด
 ```php start.php stop```

### รีสตาร์ท
 ```php start.php restart```

### รีโหลด
 ```php start.php reload```

### ตรวจสอบสถานะ
 ```php start.php status```
 
### ตรวจสอบสถานะการเชื่อมต่อ (ต้องใช้ Workerman เวอร์ชั่น >=3.5.0)
```php start.php connections```



## ความแตกต่างของ debug และ daemon mode

1. เมื่อเริ่มด้วย debug mode การพิมพ์รหัส echo、var_dump หรือ print จะแสดงผลออกทางช่องเอาท์ท์โพรมท์เอิก.

2. เมื่อเริ่มด้วย daemon mode การพิมพ์รหัส echo、var_dump หรือ print จะถูกเรีร็งไปที่ไฟล์ /dev/null และสามารถทำการตั้งค่าได้โดยการกำหนด ```Worker::$stdoutFile = '/your/path/file';```.

3. เมื่อเริ่มด้วย debug mode เมื่อทำการปิดทางเอาท์ท์โพรมท์อย่างง่าย workerman ก็จะทำการปิดและออกไปไปด้วย.

4. เมื่อเริ่มด้วย daemon mode เมื่อทำการปิดทางเอาท์ท์โพรมท์ workerman จะกิ้งฟ้อนง่ายๆและทำงานไปต่อ.

## การรีโหลดอย่างเรียบร้อยหมาะ?

ดูข้อมูลที่[原理ของการรีโหลดอย่างเรียบร้อย](../faq/reload-principle.md)

