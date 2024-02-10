البروتوكول النصي
>حدد Workerman نوعًا من البروتوكول النصي يُسمى "النص"، حيث يكون تنسيق البروتوكول عبارة عن "حزمة بيانات + رمز تغيير السطر"، أي إضافة رمز تغيير السطر في نهاية كل حزمة بيانات لتمثيل نهاية الحزمة.

على سبيل المثال، تحتوي سلسلتي buffer1 وbuffer2 أدناه على بروتوكول النص:

```php
// النص مع تغيير السطر
$buffer1 = 'abcdefghijklmn
';
// في PHP، يُمثل \n في الاقتباس المزدوج رمز تغيير السطر مثل "\n"
$buffer2 = '{"type":"say", "content":"hello"}'."\n";

// إنشاء اتصال socket مع الخادم
$client = stream_socket_client('tcp://127.0.0.1:5678');
// إرسال بيانات buffer1 ببروتوكول النص
fwrite($client, $buffer1);
// إرسال بيانات buffer2 ببروتوكول النص
fwrite($client, $buffer2);
```

بروتوكول النص بسيط للغاية وسهل الاستخدام. إذا كان المطور بحاجة إلى بروتوكول خاص به، مثل نقل البيانات مع تطبيق الجوال أو التواصل مع الأجهزة الصلبة وما إلى ذلك، فيمكنه النظر في استخدام بروتوكول النص نظرًا لسهولة تطويره وتصحيح الأخطاء.

**تصحيح بروتوكول النص**
>يمكن استخدام بروتوكول النص للتصحيح باستخدام عميل telnet، كما في المثال التالي:

قم بإنشاء ملف جديد باسم test.php

```php
require_once __DIR__ . '/Workerman/Autoloader.php';
use Workerman\Worker;

$text_worker = new Worker("text://0.0.0.0:5678");

$text_worker->onMessage =  function($connection, $data)
{
    var_dump($data);
    $connection->send("hello world");
};

Worker::runAll();
```

قم بتنفيذ `php test.php start` سيظهر ما يلي:

```text
php test.php start
Workerman[test.php] start in DEBUG mode
----------------------- WORKERMAN -----------------------------
Workerman version:3.2.7          PHP version:5.4.37
------------------------ WORKERS -------------------------------
user          worker        listen                         processes status
root          none          myTextProtocol://0.0.0.0:5678   1         [OK]
----------------------------------------------------------------
Press Ctrl-C to quit. Start success.
```

افتح نافذة الطرفية مرة أخرى واستخدم telnet للتجربة (مُستحسَن باستخدام نظام telnet في Linux)

في حالة الاختبار على الجهاز المحلي،
قم بتنفيذ telnet 127.0.0.1 5678
ثم أدخل hi واضغط على مفتاح الإدخال
سيتم استقبال الرد hello world\n

```text
telnet 127.0.0.1 5678
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
hi
hello world

```
