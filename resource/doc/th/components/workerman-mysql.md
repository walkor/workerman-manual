# Workerman/MySQL

## คำแนะนำ
โปรแกรมที่คงอยู่ในหน่วยความจำสูงจะพบปัญหา "mysql gone away" ขณะใช้ mysql โดยคำแนะนี้จะเกิดจากการที่โปรแกรมและการเชื่อมต่อกับ mysql ไม่ได้สื่อสารกันเป็นเวลานาน การเชื่อมต่อจึงถูกเซิร์ฟเวอร์ mysql ตัดออก คล้ายกับนั้นคลาสฐานข้อมูลนี้สามารถแก้ไขปัญหานี้ โดยที่เมื่อเกิด "mysql gone away" ข้อผิดพลาดจะทำการลองเชื่อมต่ออีกครั้งโดยอัตโนมัติ

## การเชื่อมต่อที่ต้องการ
คลาส mysql นี้ขึ้นอยู่กับสองส่วนขยาย [pdo](https://php.net/manual/zh/book.pdo.php) และ [pdo_mysql](https://php.net/manual/zh/ref.pdo-mysql.php) การขาดส่วนขยายจะทำให้เกิดข้อผิดพลาด "Undefined class constant 'MYSQL_ATTR_INIT_COMMAND' in ...."

การรันคำสั่ง `php -m` จะแสดงรายการขยายทุกตัวที่ติดตั้งบน php cli หากขาดส่วนขยาย pdo หรือ pdo_mysql กรุณาทำการติดตั้งเอง
 
**ระบบ centos**
 
PHP5.x
```
yum install php-pdo
yum install php-mysql
```
 
PHP7.x
```
yum install php70w-pdo_dblib.x86_64
yum install php70w-mysqlnd.x86_64
```
หากไม่พบชื่อแพ็คเกจโปรดลองใช้คำสั่ง `yum search php mysql`

**ระบบ ubuntu/debian**
 
PHP5.x
```
apt-get install php5-mysql
```
 
PHP7.x
```
apt-get install php7.0-mysql
```
ถ้าหากไม่พบชื่อแพ็คเกจโปรดลองใช้คำสั่ง `apt-cache search php mysql`

**ไม่สามารถติดตั้งได้?**
 
หากวิธีการข้างต้นไม่สามารถทำการติดตั้ง กรุณาดูที่[คู่มือ workerman-ภาคผนวก-การติดตั้งส่วนขยาย-วิธีการที่สามที่ติดตั้งจากรหัส](../appendices/install-extension.md)

## การติดตั้ง Workerman/MySQL
**วิธีที่1：**

คุณสามารถทำการติดตั้งผ่าน composer โดยการรันคำสั่งต่อไปนี้ (แต่สามารถเชื่อมต่อไม่ได้ เนื่องจาก ไทยต้วมส์ไม่เปิด)

``` 
composer require workerman/mysql
```

เมื่อคำสั่งด้านบนเสร็จสมบูรณ์ จะสร้างโฟลเดอร์ "vendor" และให้ส่งเข้าไปที่ autoload.php ภายในโปรเจค
```php
require_once __DIR__ . 'มันมี /vendor/autoload.php';
```

**วิธีที่2：**

[ดาวน์โหลด source code](https://github.com/walkor/mysql/archive/master.zip) และปลดบีบออกมาที่โฟลเดอร์โปรเจคของคุณ (ที่อยู่อิสระ) และ require ไฟล์ที่เกิดขึ้น
```php
require_once '/your/path/of/mysql-master/src/Connection.php';
```

## ข้อแนะนำ
จะแนะนำอย่างมากที่จะทำการเชื่อมต่อฐานข้อมูลใน onWorkerStart callback เพื่อหลีกเลี่ยงการเริ่มต้นการเชื่อมต่อก่อนที่จะมีการทำงาน `Worker::runAll();` เชื่อมต่อที่ถูกเริ่มจากก่อนการทำงาน `Worker::runAll();` จะอย่ในกระบวนการหลัก ด้วยการทำงานบล๋อรสายเปล่า จะทำให้เกิดความผิดพลาดเมื่อบล็อคลี่กันใช้่ช่วย

## ตัวอย่าง
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    // จัดเก็บการเชื่อมต่อ db ไว้ที่ตัวแปร global (ก็สามารถจัดเก็บไว้ที่สมาชิกสถิตในคลาสของตัวเองได้)
    global $db;
    $db = new \Workerman\MySQL\Connection('host', 'port', 'user', 'password', 'db_name');
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // ดึงการเชื่อมต่อ db ไปใช้ที่ตัวแปร global
    global $db;
    // ประมวลผล SQL
    $all_tables = $db->query('show tables');
    $connection->send(json_encode($all_tables));
};
// รัน worker
Worker::runAll();
```
## วิธีการใช้ MySQL/Connection โดยละเอียด
```php
// เชื่อมต่อฐานข้อมูล
$db = new \Workerman\MySQL\Connection('host', 'port', 'user', 'password', 'db_name');

// ดึงข้อมูลทั้งหมด
$db->select('ID,Sex')->from('Persons')->where('sex= :sex AND ID = :id')->bindValues(array('sex'=>'M', 'id' => 1))->query();
//เทียบเท่ากับ
$db->select('ID,Sex')->from('Persons')->where("sex= 'M' AND ID = 1")->query();
//เทียบเท่ากับ
$db->query("SELECT ID,Sex FROM `Persons` WHERE sex='M' AND ID = 1");

// ดึงข้อมูลแถวเดียว
$db->select('ID,Sex')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->row();
//เทียบเท่ากับ
$db->select('ID,Sex')->from('Persons')->where("sex= 'M' ")->row();
//เทียบเท่ากับ
$db->row("SELECT ID,Sex FROM `Persons` WHERE sex='M'");

// ดึงข้อมูลคอลัมน์เดียว
$db->select('ID')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->column();
//เทียบเท่ากับ
$db->select('ID')->from('Persons')->where("sex= 'F' ")->column();
//เทียบเท่ากับ
$db->column("SELECT `ID` FROM `Persons` WHERE sex='M'");

// ดึงค่าเดี่ยว
$db->select('ID')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->single();
//เทียบเท่ากับ
$db->select('ID')->from('Persons')->where("sex= 'F' ")->single();
//เทียบเท่ากับ
$db->single("SELECT ID FROM `Persons` WHERE sex='M'");

// คิวรี่ที่ซับซ้อน
$db->select('*')->from('table1')->innerJoin('table2','table1.uid = table2.uid')->where('age > :age')->groupBy(array('aid'))->having('foo="foo"')->orderByASC/*orderByDESC*/(array('did'))
->limit(10)->offset(20)->bindValues(array('age' => 13));
// เทียบเท่ากับ
$db->query('SELECT * FROM `table1` INNER JOIN `table2` ON `table1`.`uid` = `table2`.`uid`
WHERE age > 13 GROUP BY aid HAVING foo="foo" ORDER BY did LIMIT 10 OFFSET 20');

// แทรกข้อมูล
$insert_id = $db->insert('Persons')->cols(array(
    'Firstname'=>'abc',
    'Lastname'=>'efg',
    'Sex'=>'M',
    'Age'=>13))->query();
เทียบเท่ากับ
$insert_id = $db->query("INSERT INTO `Persons` ( `Firstname`,`Lastname`,`Sex`,`Age`)
VALUES ( 'abc', 'efg', 'M', 13)");

// ปรับปรุง
$row_count = $db->update('Persons')->cols(array('sex'))->where('ID=1')
->bindValue('sex', 'F')->query();
// เทียบเท่ากับ
$row_count = $db->update('Persons')->cols(array('sex'=>'F'))->where('ID=1')->query();
// เทียบเท่ากับ
$row_count = $db->query("UPDATE `Persons` SET `sex` = 'F' WHERE ID=1");

// ลบ
$row_count = $db->delete('Persons')->where('ID=9')->query();
// เทียบเท่ากับ
$row_count = $db->query("DELETE FROM `Persons` WHERE ID=9");

// ธุรกรรม
$db->beginTrans();
....
$db->commitTrans(); // หรือ $db->rollBackTrans();
```
