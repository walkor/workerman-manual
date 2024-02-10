# الخطوة الأساسية
(مثال: سيرفر بسيط لغرفة دردشة عبر Websocket)

#### 1. انشاء مسار المشروع في أي مكان
مثل: SimpleChat/
انتقل الى المسار وقم بتنفيذ `composer require workerman/workerman`

#### 2. استيراد `vendor/autoload.php` (بعد تثبيت composer)
أنشئ ملف start.php ، واستورد `vendor/autoload.php` 
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';
```

#### 3. اختيار البروتوكول
هنا اخترنا بروتوكول النص (بروتوكول مخصص في Workerman، والتنسيق يكون نصي + سطر جديد)

(حاليًا، يدعم Workerman بروتوكولات HTTP و Websocket و Text نصية، إذا كنت بحاجة إلى استخدام بروتوكول آخر، يرجى الرجوع إلى فصل البروتوكول لتطوير بروتوكول خاص بك)

#### 4. كتابة ملف البداية والتشغيل وفقًا للحاجة
على سبيل المثال، يلي ملف الدخول البسيط لغرفة الدردشة.

SimpleChat/start.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$global_uid = 0;

// عندما يتصل العميل، قم بتخصيص uid واحتفظ بالاتصال وأبلغ جميع العملاء
function handle_connection($connection)
{
    global $text_worker, $global_uid;
    // قم بتخصيص uid لهذا الاتصال
    $connection->uid = ++$global_uid;
}

// عندما يرسل العميل رسالة، قم بتوجيهها للجميع
function handle_message(TcpConnection $connection, $data)
{
    global $text_worker;
    foreach($text_worker->connections as $conn)
    {
        $conn->send("user[{$connection->uid}] said: $data");
    }
}

// عند فصل العميل، قم ببثها لكل العملاء
function handle_close($connection)
{
    global $text_worker;
    foreach($text_worker->connections as $conn)
    {
        $conn->send("user[{$connection->uid}] logout");
    }
}
// قم بإنشاء Worker ببروتوكول النص واستمع الى البورت 2347
$text_worker = new Worker("text://0.0.0.0:2347");

// قم بتشغيل عملية واحدة فقط، بهذا الشكل سيكون من السهل نقل البيانات بين العملاء
$text_worker->count = 1;

$text_worker->onConnect = 'handle_connection';
$text_worker->onMessage = 'handle_message';
$text_worker->onClose = 'handle_close';

Worker::runAll();
```

#### 5. الاختبار
يمكنك اختبار بروتوكول النص باستخدام أمر telnet
```shell
telnet 127.0.0.1 2347
```
