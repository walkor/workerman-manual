# فتح منفذ 843 لـ Flash

عندما يقوم فلاش بإجراء اتصال بمخدم الخدمة عن بعد عبر مأخذ الشبكة ، فسيُطلب ملف سياسة أمان من المنفذ 843 المقابل أولاً. وإلا فإنه لن يتمكن فلاش من إقامة الاتصال مع مخدم الخدمة. يمكن استخدام الكود التالي في Workerman لفتح منفذ 843 وإرجاع ملف سياسة الأمان.

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$flash_policy = new Worker('tcp://0.0.0.0:843');
$flash_policy->onMessage = function(TcpConnection $connection, $message)
{
    $connection->send('<?xml version="1.0"?><cross-domain-policy><site-control permitted-cross-domain-policies="all"/><allow-access-from domain="*" to-ports="*"/></cross-domain-policy>'."\0");
};

if(!defined('GLOBAL_START'))
{
    Worker::runAll();
}
```

يمكنك تخصيص محتوى سياسة الأمان XML حسب الاحتياجات الخاصة بك.
