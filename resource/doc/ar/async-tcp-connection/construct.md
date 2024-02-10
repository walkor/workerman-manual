# __construct الدالة
```php
void AsyncTcpConnection::__construct(string $remote_address, $context_option = null)
```
إنشاء كائن اتصال غير متزامن.

يمكن لـ AsyncTcpConnection لـ Workerman أن يسمح للخادم بالاتصال بشكل غير متزامن بخادم بعيد كعميل وإرسال واستقبال البيانات بشكل غير متزامن عبر واجهة send ودالة رد الاتصال onMessage.

## المعلمات
المعلمة: `remote_address`

عنوان الاتصال، مثال:
``` tcp://www.baidu.com:80 ```
``` ssl://www.baidu.com:443 ```
``` ws://echo.websocket.org:80 ```
``` frame://192.168.1.1:8080 ```
``` text://192.168.1.1:8080 ```

المعلمة: `context_option`

``` (context_option)` هذا المعلمة يتطلب (workerman >= 3.3.5) ```

تُستخدم لضبط سياق الاتصال، مثل ضبط `bindto` لتحديد أي عنوان IP (بطاقة الشبكة) ومنفذ يتم استخدامها للوصول إلى الشبكة الخارجية، وضبط شهادة SSL، إلخ.

يرجى الرجوع إلى[stream_context_create](https://php.net/manual/en/function.stream-context-create.php) و[خيارات سياق المأخذ](https://php.net/manual/zh/context.socket.php) و[خيارات سياق SSL](https://php.net/manual/zh/context.ssl.php).

## ملاحظات
حاليًا يدعم AsyncTcpConnection البروتوكولات [tcp](https://baike.baidu.com/subview/32754/8048820.htm) و [ssl](https://baike.baidu.com/view/525499.htm) و [ws](appendices/about-ws.md) و [frame](appendices/about-frame.md) و [text](appendices/about-text.md).

كما يدعم البروتوكول المخصص، راجع [كيفية تخصيص البروتوكول](../protocols/how-protocols.md)

يتطلب بروتوكول [ssl](https://baike.baidu.com/view/525499.htm) Workerman>=3.3.4 وتثبيت إضافة [openssl](https://php.net/manual/zh/book.openssl.php).

حاليًا لا يدعم AsyncTcpConnection بروتوكول [http](https://baike.baidu.com/view/9472.htm).

يمكنك استخدام `new AsyncTcpConnection('ws://...')` لإجراء اتصال بمزود خادم WebSocket عن بعد في Workerman مثل المتصفح، راجع [المثال](../appendices/about-ws.md). ولكن لا يمكنك استخدام `new AsyncTcpConnection('websocket://...')` لإجراء اتصال ببروتوكول WebSocket في Workerman.

## الأمثلة
### مثال 1: الوصول غير المتزامن إلى خدمة HTTP خارجية
```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// عند بدء تشغيل العملية، قم بإنشاء كائن اتصال بـ `www.baidu.com` بشكل غير متزامن وإرسال البيانات واستقبالها
$task->onWorkerStart = function($task)
{
    // غير مدعومة بشكل مباشر، لكن يمكن استخدام البروتوكول tcp لنمذجة بروتوكول HTTP لإرسال البيانات
    $connection_to_baidu = new AsyncTcpConnection('tcp://www.baidu.com:80');
    // عندما يتم بناء الاتصال بنجاح، قم بإرسال بيانات الطلب HTTP
    $connection_to_baidu->onConnect = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "connect success\n";
        $connection_to_baidu->send("GET / HTTP/1.1\r\nHost: www.baidu.com\r\nConnection: keep-alive\r\n\r\n");
    };
    $connection_to_baidu->onMessage = function(AsyncTcpConnection $connection_to_baidu, $http_buffer)
    {
        echo $http_buffer;
    };
    $connection_to_baidu->onClose = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "connection closed\n";
    };
    $connection_to_baidu->onError = function(AsyncTcpConnection $connection_to_baidu, $code, $msg)
    {
        echo "Error code:$code msg:$msg\n";
    };
    $connection_to_baidu->connect();
};

// تشغيل الخادم
Worker::runAll();
```

### مثال 2: الوصول غير المتزامن إلى خدمة WebSocket خارجية وضبط أي عنوان IP ومنفذ محلي للوصول
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // ضبط عنوان IP ومنفذ عنوان الجهاز المستهدف للوصول (كل اتصال بمأخذ يحتل منفذًا محليًا)
    $context_option = array(
        'socket' => array(
            // يجب أن يكون عنوان IP عنوان بطاقة الشبكة المحلي ويمكن الوصول إلى الجهة الأخرى، وإلا لا يكون صالحًا
            'bindto' => '114.215.84.87:2333',
        ),
    );

    $con = new AsyncTcpConnection('ws://echo.websocket.org:80', $context_option);

    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

### مثال 3: الوصول غير المتزامن إلى منفذ wss الخارجي وضبط شهادة SSL محلية
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // ضبط عنوان IP ومنفذ عنوان الجهاز المستهدف وكذلك شهادة SSL المحلية
    $context_option = array(
        'socket' => array(
            // يجب أن يكون عنوان IP عنوان بطاقة الشبكة المحلي ويمكن الوصول إلى الجهة الأخرى، وإلا لا يكون صالحًا
            'bindto' => '114.215.84.87:2333',
        ),
        // خيارات SSL، راجع https://php.net/manual/zh/context.ssl.php
        'ssl' => array(
            // مسار الشهادة المحلية. يجب أن يكون بتنسيق PEM ويتضمن الشهادة المحلية والمفتاح الخاص.
            'local_cert'        => '/your/path/to/pemfile',
            // كلمة مرور ملف local_cert.
            'passphrase'        => 'your_pem_passphrase',
            // هل يُسمح بالشهادة الموقعة ذاتيًا.
            'allow_self_signed' => true,
            // هل هناك حاجة للتحقق من شهادة SSL.
            'verify_peer'       => false
        )
    );

    // الاتصال الغير متزامن
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443', $context_option);

    // ضبط الوصول بطريقة التشفير باستخدام SSL
    $con->transport = 'ssl';

    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```
