# مثال بسيط للتطوير

## التثبيت

**تثبيت workerman**
استخدم الأمر التالي في مجلد فارغ
`composer require workerman/workerman`

## المثال الأول: تقديم خدمة الويب باستخدام بروتوكول HTTP
**إنشاء ملف start.php**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

// إنشاء Worker يستمع على المنفذ 2345، ويستخدم بروتوكول http
$http_worker = new Worker("http://0.0.0.0:2345");

// تشغيل ٤ عمليات لتقديم الخدمة للعملاء
$http_worker->count = 4;

// عند استقبال البيانات المرسلة من المتصفح، ترد بكلمة "hello world" للمتصفح
$http_worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // إرسال كلمة "hello world" إلى المتصفح
    $connection->send('hello world');
};

// تشغيل الـ worker
Worker::runAll();
```

**تشغيل من خلال سطر الأوامر (لمستخدمي ويندوز استخدام [سطر الأوامر cmd](https://baike.baidu.com/item/%E5%91%BD%E4%BB%A4%E6%8F%90%E7%A4%BA%E7%AC%A6?fromtitle=CMD&fromid=1193011&type=syn)، وهكذا)**
```shell
php start.php start
```

**الاختبار**

فرض أن عنوان IP الخادم هو 127.0.0.1

افتح المتصفح واستخدم الرابط http://127.0.0.1:2345

 **ملاحظة:**

1 - إذا حدثت مشكلة في الوصول، يُرجى الرجوع إلى القسم [أسباب فشل اتصال العميل](../faq/client-connect-fail.md) للتحقق.

2 - الخادم يستخدم بروتوكول http فقط ولا يمكن التواصل مباشرة باستخدام بروتوكول WebSocket أو أي بروتوكول آخر.

## المثال الثاني: تقديم خدمة باستخدام بروتوكول WebSocket
**إنشاء ملف ws_test.php**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// ملاحظة: هنا يختلف الأمر عن المثال السابق، إذ يتم استخدام بروتوكول WebSocket
$ws_worker = new Worker("websocket://0.0.0.0:2000");

// تشغيل ٤ عمليات لتقديم الخدمة للعملاء
$ws_worker->count = 4;

// عند استقبال بيانات من العميل، يتم إرجاع كلمة "hello $data" للعميل
$ws_worker->onMessage = function(TcpConnection $connection, $data)
{
    // إرسال "hello $data" للعميل
    $connection->send('hello ' . $data);
};

// تشغيل الـ worker
Worker::runAll();
```

**تشغيل من خلال سطر الأوامر**
```shell
php ws_test.php start
```

**الاختبار**

افتح متصفح Chrome، ثم استخدم F12 لفتح لوحة التحكم، ثم استخدم الكونسول لإدخال (أو قم بتضمين الكود التالي في صفحة HTML وتشغيله باستخدام JavaScript)

```javascript
// فرض أن عنوان IP الخادم هو 127.0.0.1
ws = new WebSocket("ws://127.0.0.1:2000");
ws.onopen = function() {
    alert("تم الاتصال بنجاح");
    ws.send('tom');
    alert("تم إرسال سلسلة نصية إلى الخادم: tom");
};
ws.onmessage = function(e) {
    alert("تم استقبال رسالة من الخادم: " + e.data);
};
```

  **ملاحظة:**

1 - إذا حدثت مشكلة في الوصول، يُرجى الرجوع إلى [قسم الأسئلة الشائعة - فشل اتصال العميل](../faq/client-connect-fail.md) للتحقق.

2 - الخادم يستخدم بروتوكول WebSocket فقط ولا يمكن التواصل مباشرة باستخدام بروتوكول HTTP أو أي بروتوكول آخر.

## المثال الثالث: الاتصال مباشرة باستخدام TCP لنقل البيانات
**إنشاء ملف tcp_test.php**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// إنشاء Worker يستمع على المنفذ 2347، ولا يستخدم أي بروتوكول طبقة تطبيقية
$tcp_worker = new Worker("tcp://0.0.0.0:2347");

// تشغيل ٤ عمليات لتقديم الخدمة للعملاء
$tcp_worker->count = 4;

// عندما يصل البيانات من العميل
$tcp_worker->onMessage = function(TcpConnection $connection, $data)
{
    // إرسال "hello $data" للعميل
    $connection->send('hello ' . $data);
};

// تشغيل الـ worker
Worker::runAll();
```

**تشغيل من خلال سطر الأوامر**

```shell
php tcp_test.php start
```

**الاختبار: تشغيل من خلال سطر الأوامر**
(النتائج التالية تظهر في سطر الأوامر لنظام Linux، وتختلف قليلا في نظام Windows)
```shell
telnet 127.0.0.1 2347
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
tom
hello tom
```

**ملاحظة:**

1 - إذا حدثت مشكلة في الوصول، يُرجى الرجوع إلى [قسم الأسئلة الشائعة - فشل اتصال العميل](../faq/client-connect-fail.md) للتحقق.

2 - الخادم يستخدم بروتوكول TCP فقط ولا يمكن التواصل مباشرة باستخدام بروتوكول WebSocket أو بروتوكول HTTP أو أي بروتوكول آخر.
