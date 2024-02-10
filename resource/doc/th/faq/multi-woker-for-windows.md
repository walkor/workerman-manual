วินโดว์เครื่องมือไม่สามารถใช้งาน Worker หลายตัวในไฟล์ PHP เดียวกันได้

เช่นในไฟล์ test.php ด้านล่าง
```php
<?php
...
$socket_server = new Worker("tcp://0.0.0.0:5555");
$socket_server->on....
$websocket_server = new Worker("websocket://0.0.0.0:6666");
$websocket_server->on....
$webserver = new WebServer("http://0.0.0.0:6666");
$webserver->addRoot(...
...
```
การเริ่มต้นบน windows จะแจ้งเตือนคำข้อผิดพลาด
```multi workers init in one php file are not support```

## วิธีแก้ไข
วิธีแก้ไขคือการใช้สคริปต์เริ่มต้นหลายๆ รายการ โดยการดำเนินการบนแต่ละฟังก์ชันหรือบนพอร์ตเดียวกัน

ถ้าหากต้องการเริ่มต้น Worker 2 ตัว (tcp และ websocket) และ WebServer ให้สร้างสคริปต์เริ่มต้นทั้งหมด 3 ไฟล์ เช่น start\_socket\_server.php และ start\_websocket\_server.php และ start\_webserver.php

ตัวอย่างเช่น:

ไฟล์ start\_socket\_server.php
```php
...
$socket_server = new Worker("tcp://0.0.0.0:5555");
$socket_server->on....
....
```

ไฟล์ start\_websocket\_server.php
```php
...
$websocket_server = new Worker("websocket://0.0.0.0:6666");
$websocket_server->on....
 ...
```

ไฟล์ start\_webserver.php
```php
...
$webserver = new WebServer("http://0.0.0.0:6666");
$webserver->addRoot(...
 ...
```

เมื่อเริ่มต้น จะเป็นดังตัวอย่างข้างล่าง โดยการเริ่มต้นสามสคริปต์ (ระหว่าง windows cmd command line)

```php start_socket_server.php start_websocket_server.php start_webserver.php```
