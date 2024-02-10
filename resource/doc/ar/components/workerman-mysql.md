# ووركرمان/ماي إس كيو إل

## الشرح
في البرامج الموجودة في الذاكرة بشكل دائم، يمكن أن يتعرض البرنامج لخطأ "mysql gone away" عند استخدام mysql، ويعود هذا الخطأ إلى عدم تفاعل الاتصال بين البرنامج وخادم mysql لفترة طويلة مما يؤدى إلى فصل الاتصال من خادم mysql. يمكن للفئة قاعدة البيانات هذه حل هذه المشكلة عند حدوث خطأ "mysql gone away" سيقوم بإعادة المحاولة تلقائيًا.

## الاعتمادات
تعتمد فئة mysql هذه على الامتدادين [pdo](https://php.net/manual/zh/book.pdo.php) و[pdo_mysql](https://php.net/manual/zh/ref.pdo-mysql.php) وفي حالة عدم توفر هذين الامتدادين سيتم عرض خطأ "Undefined class constant 'MYSQL_ATTR_INIT_COMMAND' in ..." 

يمكنك تشغيل الأمر `php -m` لعرض جميع الامتدادات التي تم تثبيتها مسبقًا باستخدام php cli، إذا لم يكن pdo أو pdo_mysql موجودين، يرجى تثبيتهما بنفسك.

** نظام centos **
PHP5.x
`` `
yum install php-pdo
yum install php-mysql
```

PHP7.x
`` `
yum install php70w-pdo_dblib.x86_64
yum install php70w-mysqlnd.x86_64
```
إذا لم تجد أسماء الحزمة، جرب استخدام `yum search php mysql` للبحث

** أنظمة أوبونتو / ديبيان **
PHP5.x
`` `
apt-get install php5-mysql
```

PHP7.x
`` `
apt-get install php7.0-mysql
```

إذا لم تجد أسماء الحزمة، جرب استخدام `apt-cache search php mysql` للبحث

** هل لا يمكن تثبيتها باستخدام الطرق أعلاه؟ **

إذا لم يكن بالإمكان تثبيت الامتدادات باستخدام الطرق المذكورة أعلاه، يرجى الرجوع إلى [علامة ووركرمان - الملحق - تثبيت الامتدادات - الطريقة الثالثة تثبيت الشفرة المصدرية](../appendices/install-extension.md). 

## تثبيت ووركرمان/ماي إس كيو إل
** الطريقة 1: **

يمكنك تثبيتها باستخدام composer، قم بتشغيل الأمر التالي من خط الأوامر (مصدر composer في الخارج، قد يتأخر عملية التثبيت بشكل كبير).

`` `
composer require workerman/mysql
`` `

بعد نجاح الأمر أعلاه، سيتم إنشاء دليل vendor، ثم قم بإدراج autoload.php الموجود في الدليل في مشروعك.
`` `php
require_once __DIR__ . '/vendor/autoload.php';
`` `

** الطريقة 2: **

[قم بتنزيل الشفرة المصدرية](https://github.com/walkor/mysql/archive/master.zip)، قم بفك الضغط عن الدليل وضعه في مشروعك (الموقع غير مهم)، وقم بتضمين الملف المصدري مباشرة.

`` `php
require_once '/your/path/of/mysql-master/src/Connection.php';
`` `

## ملاحظة
نوصي بشدة بتهيئة اتصال قاعدة البيانات في الردود على onWorkerStart، تجنبًا لتهيئة الاتصال قبل تشغيل ```Worker::runAll();```، حيث يعتبر تهيئة الاتصال قبل تشغيل ```Worker::runAll();``` تابعًا للعمل الرئيسي لمنتدى المشروع، وسيقوم العمل الفرعي بالوراثة من هذا الاتصال، حيث يؤدي مشاركة العمل الرئيسي والفرعي لنفس اتصال قاعدة البيانات إلى حدوث أخطاء.

## مثال
`` `php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    // قم بتخزين مثيل db في متغير عام (يمكن أن تتم تخزينه أيضًا في عضو ثابت بالدرجة في كائن)
    global $db;
    $db = new \Workerman\MySQL\Connection('host', 'port', 'user', 'password', 'db_name');
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // نتنال مثيل db عن طريق المتغير العام
    global $db;
    // نفذ SQL
    $all_tables = $db->query('show tables');
    $connection->send(json_encode($all_tables));
};
// تشغيل الووركر
Worker::runAll();
`` `

## استخدام وظيفة اتصال ماي إس كيو إل بالتفصيل
`` `php
// تهيئة اتصال قاعدة البيانات
$db = new \Workerman\MySQL\Connection('host', 'port', 'user', 'password', 'db_name');

// الحصول على كافة البيانات
$db->select('ID,Sex')->from('Persons')->where('sex= :sex AND ID = :id')->bindValues(array('sex'=>'M', 'id' => 1))->query();
//مماثلة ل
$db->select('ID,Sex')->from('Persons')->where("sex= 'M' AND ID = 1")->query();
//مماثلة ل
$db->query("SELECT ID,Sex FROM `Persons` WHERE sex='M' AND ID = 1");


// الحصول على صف واحد من البيانات
$db->select('ID,Sex')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->row();
//مماثلة ل
$db->select('ID,Sex')->from('Persons')->where("sex= 'M' ")->row();
//مماثلة ل
$db->row("SELECT ID,Sex FROM `Persons` WHERE sex='M'");


// الحصول على عمود واحد من البيانات
$db->select('ID')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->column();
//مماثلة ل
$db->select('ID')->from('Persons')->where("sex= 'F' ")->column();
//مماثلة ل
$db->column("SELECT `ID` FROM `Persons` WHERE sex='M'");

// الحصول على قيمة واحدة
$db->select('ID')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->single();
//مماثلة ل
$db->select('ID')->from('Persons')->where("sex= 'F' ")->single();
//مماثلة ل
$db->single("SELECT ID FROM `Persons` WHERE sex='M'");

// الاستعلام المعقد
$db->select('*')->from('table1')->innerJoin('table2','table1.uid = table2.uid')->where('age > :age')->groupBy(array('aid'))->having('foo="foo"')->orderByASC/*orderByDESC*/(array('did'))
->limit(10)->offset(20)->bindValues(array('age' => 13));
// مماثلة ل
$db->query('SELECT * FROM `table1` INNER JOIN `table2` ON `table1`.`uid` = `table2`.`uid`
WHERE age > 13 GROUP BY aid HAVING foo="foo" ORDER BY did LIMIT 10 OFFSET 20');

// إدراج
$insert_id = $db->insert('Persons')->cols(array(
    'Firstname'=>'abc',
    'Lastname'=>'efg',
    'Sex'=>'M',
    'Age'=>13))->query();
مماثلة ل
$insert_id = $db->query("INSERT INTO `Persons` ( `Firstname`,`Lastname`,`Sex`,`Age`)
VALUES ( 'abc', 'efg', 'M', 13)");

// تحديث
$row_count = $db->update('Persons')->cols(array('sex'))->where('ID=1')
->bindValue('sex', 'F')->query();
// مماثلة ل
$row_count = $db->update('Persons')->cols(array('sex'=>'F'))->where('ID=1')->query();
// مماثلة ل
$row_count = $db->query("UPDATE `Persons` SET `sex` = 'F' WHERE ID=1");

// حذف
$row_count = $db->delete('Persons')->where('ID=9')->query();
// مماثلة ل
$row_count = $db->query("DELETE FROM `Persons` WHERE ID=9");

// المعاملة
$db->beginTrans();
....
$db->commitTrans(); // or $db->rollBackTrans();

```
