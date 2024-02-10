# เมธอด connect

```php
void AsyncTcpConnection::connect()
```
ดำเนินการเชื่อมต่อแบบไม่ซิงโครนัส ซึ่งเมธอดนี้จะส่งคืนทันที

โปรดทราบ: หากต้องการตั้งค่า onError callback สำหรับการเชื่อมต่อแบบไม่ซิงโครนัส คุณควรตั้งค่าก่อนที่ connect จะถูกเรียก ถ้าไม่จะสามารถทำให้ onError callback ไม่ทำงานได้ เช่น ในตัวอย่างด้านล่าง onError callback อาจไม่ทำงาน ทำให้ไม่สามารถจับเหตุการณ์ล้มเหลวของการเชื่อมต่อแบบไม่ซิงโครนัสได้

```php
$connection = new AsyncTcpConnection('tcp://baidu.com:81');
// ทำการเชื่อมต่อโดยยังไม่ได้ตั้งค่า onError callback
$connection->connect();
$connection->onError = function($connection, $err_code, $err_msg)
{
    echo "$err_code, $err_msg";
};
```

### พารามิเตอร์
ไม่มีพารามิเตอร์

### คืนค่า
ไม่มีค่าที่ส่งคืน

### ตัวอย่าง พร็อกซี่ MySQL

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// ที่อยู่จริงของ mysql สมมติว่าเป็นพอร์ต 3306 บนเครื่องนี้
$REAL_MYSQL_ADDRESS = 'tcp://127.0.0.1:3306';

// พร็อกซี่จะฟังการ์ที่พอร์ต 4406 บน localhost
$proxy = new Worker('tcp://0.0.0.0:4406');

$proxy->onConnect = function(TcpConnection $connection)
{
    global $REAL_MYSQL_ADDRESS;
    // เชื่อมต่อแบบไม่ซิงโครนัสไปที่เซิร์ฟเวอร์ mysql จริง
    $connection_to_mysql = new AsyncTcpConnection($REAL_MYSQL_ADDRESS);
    // เมื่อมีข้อมูลเข้ามาจากการเชื่อมต่อ mysql ให้ส่งไปยังการเชื่อมต่อของไคลเอ็นต์ที่เกี่ยวข้อง
    $connection_to_mysql->onMessage = function(AsyncTcpConnection $connection_to_mysql, $buffer)use($connection)
    {
        $connection->send($buffer);
    };
    // เมื่อการเชื่อมต่อ mysql ปิด ให้ปิดการเชื่อมต่อจริงในตัวแทนไคลเอ็นต์
    $connection_to_mysql->onClose = function(AsyncTcpConnection $connection_to_mysql)use($connection)
    {
        $connection->close();
    };
    // เมื่อการเชื่อมต่อ mysql เกิดข้อผิดพลาด ให้ปิดการเชื่อมต่อจริงในตัวแทนไคลเอ็นต์
    $connection_to_mysql->onError = function(AsyncTcpConnection $connection_to_mysql)use($connection)
    {
        $connection->close();
    };
    // ดำเนินการเชื่อมต่อแบบไม่ซิงโครนัส
    $connection_to_mysql->connect();

    // เมื่อมีข้อมูลเข้ามาจากไคลเอ็นต์ ให้ส่งไปยังการเชื่อมต่อของ mysql ที่เกี่ยวข้อง
    $connection->onMessage = function(TcpConnection $connection, $buffer)use($connection_to_mysql)
    {
        $connection_to_mysql->send($buffer);
    };
    // เมื่อการเชื่อมต่อของไคลเอ็นต์ปิด ให้ปิดการเชื่อมต่อของ mysql ที่เกี่ยวข้อง
    $connection->onClose = function(TcpConnection $connection)use($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };
    // เมื่อการเชื่อมต่อของไคลเอ็นต์เกิดข้อผิดพลาด ให้ปิดการเชื่อมต่อของ mysql ที่เกี่ยวข้อง
    $connection->onError = function(TcpConnection $connection)use($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };

};
// รันเวิร์กเกอร์
Worker::runAll();
```

 **การทดสอบ**

```sh
mysql -uroot -P4406 -h127.0.0.1 -p

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 25004
Server version: 5.5.31-1~dotdeb.0 (Debian)

Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```
