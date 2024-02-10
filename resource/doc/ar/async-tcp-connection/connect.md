يتم استخدام الدالة connect لتنفيذ عملية الاتصال الغير متزامن. سيتم إرجاع هذه الدالة فورًا.

ملاحظة: إذا كنت بحاجة إلى تعيين استدعاء onError للاتصال الغير متزامن ، يجب ذلك أن يتم ذلك قبل تنفيذ الاتصال ، وإلا فإن استدعاء onError قد لا يتم تشغيله. على سبيل المثال ، يمكن ألا يتم تشغيل استدعاء onError في المثال التالي ، وبالتالي لا يمكن التقاط حدوث اتصال غير متزامن فاشل.

```php
$connection = new AsyncTcpConnection('tcp://baidu.com:81');
// لم يتم تعيين استدعاء onError عند تنفيذ الاتصال بعد
$connection->connect();
$connection->onError = function($connection, $err_code, $err_msg)
{
    echo "$err_code, $err_msg";
};
```

### المعلمات
لا توجد معلمات

### القيمة المُرجَعة
لا توجد قيمة مرجعية

### مثال لوكيل Mysql

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// عنوان mysql الحقيقي، نفترض أنها 3306 على الجهاز المحلي
$REAL_MYSQL_ADDRESS = 'tcp://127.0.0.1:3306';

// تنصت الوكيل على المنفذ المحلي 4406
$proxy = new Worker('tcp://0.0.0.0:4406');

$proxy->onConnect = function(TcpConnection $connection)
{
    global $REAL_MYSQL_ADDRESS;
    // إنشاء اتصال غير متزامن إلى خادم mysql الفعلي
    $connection_to_mysql = new AsyncTcpConnection($REAL_MYSQL_ADDRESS);
    // عندما يصل اتصال mysql ببيانات، قم بتوجيهها إلى اتصال العميل المقابل
    $connection_to_mysql->onMessage = function(AsyncTcpConnection $connection_to_mysql, $buffer)use($connection)
    {
        $connection->send($buffer);
    };
    // عند إغلاق اتصال mysql، قم بإغلاق الاتصال بالوكيل المقابل للعميل
    $connection_to_mysql->onClose = function(AsyncTcpConnection $connection_to_mysql)use($connection)
    {
        $connection->close();
    };
    // عند حدوث خطأ في اتصال mysql، قم بإغلاق الاتصال بالوكيل المقابل للعميل
    $connection_to_mysql->onError = function(AsyncTcpConnection $connection_to_mysql)use($connection)
    {
        $connection->close();
    };
    // تنفيذ الاتصال غير المتزامن
    $connection_to_mysql->connect();

    // عندما يصل العميل بيانات، قم بتوجيهها إلى اتصال mysql المقابل
    $connection->onMessage = function(TcpConnection $connection, $buffer)use($connection_to_mysql)
    {
        $connection_to_mysql->send($buffer);
    };
    // عند فصل اتصال العميل، فصل الاتصال بmysql المقابل
    $connection->onClose = function(TcpConnection $connection)use($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };
    // عند حدوث خطأ في اتصال العميل، قم بإغلاق الاتصال بmysql المقابل
    $connection->onError = function(TcpConnection $connection)use($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };

};
// تشغيل الوركر
Worker::runAll();
``` 

 **اختبار**

``` 
mysql -uroot -P4406 -h127.0.0.1 -p

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 25004
Server version: 5.5.31-1~dotdeb.0 (Debian)

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```
