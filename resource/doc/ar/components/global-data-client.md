# عميل GlobalData المكون

**``` (يتطلب إصدار Workerman> = 3.3.0) ```**

# __construct
```php
void \GlobalData\Client::__construct(mixed $server_address)
```

يقوم بتحديث كائن عميل \GlobalData\Client. يمكنك مشاركة البيانات بين العمليات عن طريق تعيين الخصائص على كائن العميل.

### المعاملات
عنوان ملقم GlobalData، بالتنسيق ```<عنوان IP>:<منفذ>```، مثال ```127.0.0.1:2207```.

إذا كان GlobalData server مجموعة، يجب تمرير مصفوفة عناوين، مثال ```array('10.0.0.10:2207', '10.0.0.0.11:2207')```

## الشرح
يدعم عمليات التعيين، والقراءة، والفحص، والإزالة.
يدعم أيضًا عمليات الـ CAS الذرية.

## مثال

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// ملقم GlobalData
$global_worker = new GlobalData\Server('0.0.0.0', 2207);

$worker = new Worker('tcp://0.0.0.0:6636');
// عند بدء العملية
$worker->onWorkerStart = function()
{
    // بادئ ذي بدء عميل بيانات عام global
    global $global;
    $global = new \GlobalData\Client('127.0.0.1:2207');
};
// عند كل مرة يتم فيها استقبال رسالة من الخادم
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // تغيير قيمة $global->somedata وسيشترك العمليات الأخرى في هذا المتغير $global->somedata
    global $global;
    echo "now global->somedata=".var_export($global->somedata, true)."\n";
    echo "set \$global->somedata=$data";
    $global->somedata = $data;
};
Worker::runAll();
```

### جميع الاستخدامات (يمكن أن تستخدم أيضًا في بيئة php-fpm)
```php
require_once __DIR__ . '/vendor/autoload.php';

$global = new Client('127.0.0.1:2207');

var_export(isset($global->abc));

$global->abc = array(1,2,3);

var_export($global->abc);

unset($global->abc);

var_export($global->add('abc', 10));

var_export($global->increment('abc', 2));

var_export($global->cas('abc', 12, 18));

```

## ملاحظة:
لا يمكن مشاركة أنواع البيانات ذات المصادر مثل اتصالات MySQL أو اتصالات المقابس باستخدام مكون GlobalData.

إذا كنت تستخدم GlobalData/Client في بيئة Workerman، يجب أن تقوم بتحديث كائن GlobalData/Client في الردود onXXX، على سبيل المثال، يجب أن تقوم بتحديثه في onWorkerStart.

لن يعمل التعامل مع المتغيرات المشتركة بهذه الطريقة.
```php
$global->somekey = array();
$global->somekey[]='xxx';

$global->someObject = new someClass();
$global->someObject->someVar = 'xxx';
```
يمكنك فعل ذلك بهذه الطريقة
```php
$somekey = array();
$somekey[] = 'xxx';
$global->somekey = $somekey;

$someObject = new someClass();
$someObject->someVar = 'xxx';
$global->someObject = $someObject;
```
