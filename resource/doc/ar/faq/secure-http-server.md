# إنشاء خدمة HTTPS

**السؤال:**

كيف يمكن لـ Workerman إنشاء خدمة HTTPS بحيث يمكن للعملاء الاتصال بالخادم عبر بروتوكول HTTPS.

**الجواب:**

البروتوكول HTTPS في الواقع هو بروتوكول HTTP مع إضافة SSL، أي أنه يتم إضافة طبقة SSL إلى بروتوكول HTTP. يدعم Workerman بروتوكول HTTP وكذلك SSL (```يلزم إصدار Workerman >=3.3.7```). لذا، يكفي فتح SSL على أساس بروتوكول HTTP لدعم بروتوكول HTTPS.

هناك حلان شائعان لجعل Workerman يدعم HTTPS. الأول هو فتح SSL مباشرة، والآخر هو استخدام Nginx كوكيل SSL. يمكن اختيار أحد الحلين، ولا يمكن تعيينهما معًا معًا.

## فتح SSL في Workerman

**التحضير:**

1. إصدار Workerman >=3.3.7
2. تثبيت امتداد OpenSSL في PHP
3. لقد حصلت بالفعل على شهادة (ملف pem/crt ومفتاح) ووضعتها في /etc/nginx/conf.d/ssl

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// الشهادة يفضل أن تكون شهادة صالحة
$context = array(
    'ssl' => array(
        'local_cert'        => '/etc/nginx/conf.d/ssl/server.pem', // يمكن أيضًا أن تكون ملف crt
        'local_pk'          => '/etc/nginx/conf.d/ssl/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // إذا كانت الشهادة ذاتية التوقيع فيجب تمكين هذا الخيار
    )
);
// هنا يتم تحديد بروتوكول HTTP
$worker = new Worker('http://0.0.0.0:443', $context);
// تعيين نقل البيانات لفتح SSL، ليصبح بروتوكول HTTP+SSL أي HTTPS
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

من خلال الكود أعلاه تم إنشاء خدمة HTTPS، وبالتالي يمكن للعملاء الاتصال بWorkerman عبر بروتوكول HTTPS لتحقيق التواصل الآمن والمشفر.

**الاختبار:**

أدخل عنوان URL ```https://domain:443``` في شريط عنوان المتصفح.

**ملاحظات:**

1. يجب أن يتم الاتصال بمنفذ HTTPS عبر بروتوكول HTTPS ولا يمكن الاتصال عبر بروتوكول HTTP.
2. عادة، يكون الشهادة مرتبطة بالنطاق، لذا يرجى استخدام النطاق أثناء الاختبار، وعدم استخدام عنوان IP.
3. إذا كان الاتصال ببروتوكول HTTPS غير متاح، يرجى التحقق من جدار الحماية على الخادم.

## استخدام Nginx كوكيل SSL

بالإضافة إلى استخدام SSL الخاص بـ Workerman، يمكن أيضًا استخدام Nginx كوكيل لتحقيق HTTPS.

> **ملاحظة** 
> يجب اختيار إما استخدام Nginx كوكيل SSL أو تعيين SSL في Workerman، ولا يمكن تشغيلهما معًا.

مبدأ التواصل والعمل هو كالتالي:

1. يبدأ العميل الاتصال بـ Nginx عبر HTTPS.
2. يقوم Nginx بتحويل البيانات ببروتوكول HTTPS إلى بروتوكول HTTP ويحولها إلى منفذ Workerman HTTP.
3. بمجرد استلام Workerman للبيانات، يتم التعامل معها من ناحية الأعمال ويتم إرجاع بيانات ببروتوكول HTTP إلى Nginx.
4. يقوم Nginx بتحويل البيانات ببروتوكول HTTP إلى HTTPS ويحولها إلى العميل.

### مرجع تكوين Nginx

**الشروط المسبقة والإعداد:**

1. نفترض أن Workerman يستمع على المنفذ 8181 (بروتوكول HTTP).
2. لدينا شهادة (ملف pem/crt ومفتاح) موجودة في /etc/nginx/conf.d/ssl.
3. نريد استخدام Nginx لفتح المنفذ 443 لتقديم خدمة wss بروتوكول (يمكن تعديل المنفذ حسب الحاجة).

**تكوين Nginx جامع كما يلي:**

```
upstream workerman {
    server 127.0.0.1:8181;
    keepalive 10240;
}


server {
  listen 443;
  server_name domain.com;
  access_log off;
  
  ssl on;
  ssl_certificate /etc/nginx/conf.d/ssl/server.pem;
  ssl_certificate_key /etc/nginx/conf.d/ssl/server.key;
  ssl_session_timeout 5m;
  ssl_session_cache shared:SSL:50m;
  ssl_protocols SSLv3 SSLv2 TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;

  location /
  {
    proxy_pass http://workerman;
    proxy_http_version 1.1;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Connection "";
  }
}
```
