# التشفير والنقل بشكل آمن - SSL/TLS

**السؤال:**

كيف يمكن ضمان أمان التواصل مع Workerman؟

**الجواب:**

أحد الطرق البسيطة هي إضافة طبقة تشفير [SSL](https://baike.baidu.com/item/ssl) فوق بروتوكول الاتصال. على سبيل المثال، بروتوكول wss و[https](https://baike.baidu.com/item/https) يعتمدان على نقل البيانات بتشفير [SSL](https://baike.baidu.com/item/ssl)، مما يجعلها آمنة للغاية. يدعم Workerman نفسه [SSL](https://baike.baidu.com/item/ssl) (```يتطلب Workerman إصدار 3.3.7 أو أحدث```)، ويكفي ضبط الخصائص لتمكين الSSL.

بالطبع، يمكن للمطورين أيضًا تصميم آلية تشفير خاصة بهم استنادًا إلى بعض خوارزميات التشفير والفك.

## كيفية تمكين SSL في Workerman:

**الخطوات الأولية:**

1. إصدار Workerman يجب أن يكون 3.3.7 أو أحدث.
2. يجب أن يكون الخادم يشغل إضافة openssl في PHP.
3. يجب أن يكون قد تم الحصول على شهادة (ملف pem/crt ومفتاح) ووضعها في /etc/nginx/conf.d/ssl.

**الكود:**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// من الأفضل أن تكون الشهادة مأخوذة من جهة موثوقة
$context = array(
    'ssl' => array(
        'local_cert'        => '/etc/nginx/conf.d/ssl/server.pem', // يمكن أن يكون ملف crt أيضًا
        'local_pk'          => '/etc/nginx/conf.d/ssl/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // إذا كانت الشهادة موقعة ذاتيًا، فهذا الخيار يجب تمكينه
    )
);
// يتم هنا تعيين بروتوكول websocket، يمكن أيضًا تغييره إلى بروتوكول http أو غيره
$worker = new Worker('websocket://0.0.0.0:443', $context);
// يتم تحديد النقل لتمكين SSL
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

## كيفية تمكين Server Name Indication (SNI) في Workerman
يمكن تحقيقها لربط عدة شهادات في نفس العنوان IP والمنفذ.

**دمج ملفات شهادة .pem و .key:**

يتم دمج محتوى كل ملف .pem مع ملف .key المقابل له، حيث يتم إضافة محتوى ملف .key إلى نهاية ملف .pem (إذا كان ملف .pem يحتوي بالفعل على المفتاح الخاص، فيمكن تجاهل هذه الخطوة).

**يرجى الملاحظة: يُدمج ملف الشهادة الفردية ومفتاحها فقط، ولا يتم دمج كافة ملفات الشهادات في ملف واحد.**

على سبيل المثال، يمكن أن يكون محتوى ملف .pem بعد الدمج على النحو التالي *host1.com.pem*:
```text
-----BEGIN CERTIFICATE-----
MIIGXTCBA...
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
MIIFBzCCA...
-----END CERTIFICATE-----
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAA....
-----END RSA PRIVATE KEY-----
```

**الكود:**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$context = array(
    'ssl' => array(
        'SNI_enabled' => true, // تمكين SNI
        'SNI_server_certs' => [ // تحديد عدة شهادات
            'host1.com' => '/path/host1.com.pem', // الشهادة 1
            'host2.com' => '/path/host2.com.pem', // الشهادة 2
        ],
        'local_cert' => '/path/default.com.pem', // الشهادة الافتراضية
        'local_pk'   => '/path/default.com.key',
    )
);
$worker = new Worker('websocket://0.0.0.0:443', $context);
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```
