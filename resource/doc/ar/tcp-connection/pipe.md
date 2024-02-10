# الأنبوب
## الوصف:
```php
void Connection::pipe(TcpConnection $target_connection)
```

## المعاملات
يقوم بتوجيه تدفق البيانات من الاتصال الحالي إلى الاتصال المستهدف. مدمج مع التحكم في حركة المرور. هذا الأسلوب مفيد جدًا لبروكسي TCP.

## مثال: بروكسي TCP

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8483');
$worker->count = 12;

// بعد إنشاء اتصال TCP
$worker->onConnect = function(TcpConnection $connection)
{
    // إنشاء اتصال غير متزامن محلي مع منفذ 80
    $connection_to_80 = new AsyncTcpConnection('tcp://127.0.0.1:80');
    // ضبط توجيه بيانات اتصال العميل الحالي إلى اتصال منفذ 80
    $connection->pipe($connection_to_80);
    // ضبط توجيه بيانات الاتصال بمنفذ 80 إلى اتصال العميل
    $connection_to_80->pipe($connection);
    // الاتصال غير المتزامن
    $connection_to_80->connect();
};

// تشغيل العامل
Worker::runAll();
```
