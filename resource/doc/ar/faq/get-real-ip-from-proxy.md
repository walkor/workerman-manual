كيفية الحصول على عنوان IP الحقيقي للعميل عن طريق الوسيط nginx/apache؟
عند استخدام nginx/apache كوكيل ل workerman ، فإن nginx/apache في الواقع يعمل كعميل ل workerman ، لذا عنوان IP العميل الذي يتم الحصول عليه في workerman هو عنوان IP لخادم nginx/apache وليس العنوان الفعلي للعميل. يمكن الاطلاع على الأساليب التالية للحصول على عنوان IP العميل الفعلي.

**المبدأ:**

يقوم nginx/apache بتمرير عنوان IP الحقيقي للعميل من خلال رأس http المرسلة، على سبيل المثال، بإضافة ```proxy_set_header X-Real-IP $remote_addr;``` في موقع التكوين nginx. يقوم workerman بقراءة قيمة هذا الرأس، ويقوم بحفظ هذه القيمة في "كائن $connection" ( يمكن لـ GatewayWorker حفظه في متغير $_SESSION) ، ويمكن قراءة هذا المتغير مباشرة عند الاستخدام.

**ملاحظة:**

هذه الإعدادات تنطبق على بروتوكولات http / https و ws / wss. بالنسبة للبروتوكولات الأخرى للحصول على عنوان IP العميل، يجب على خادم الوكيل إدراج بيانات IP في حزمة البيانات لنقل عنوان IP العميل الحقيقي.

**تكوين nginx على النحو التالي:**
```nginx
server {
  listen 443;

  ssl on;
  ssl_certificate /etc/ssl/server.pem;
  ssl_certificate_key /etc/ssl/server.key;
  ssl_session_timeout 5m;
  ssl_session_cache shared:SSL:50m;
  ssl_protocols SSLv3 SSLv2 TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;

  location /wss
  {
    proxy_pass http://127.0.0.1:8282;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
    # هذا الجزء يستخدم رأس http لنقل عنوان IP العميل الحقيقي
    proxy_set_header X-Real-IP $remote_addr;
  }
  
  # موقع / {} يحتوي على التكوينات الأخرى للموقع...
}
```

**قراءة عنوان IP العميل في workerman من الرأس المعدة من قبل nginx:**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:7272');

// عندما يتم الاتصال بالعميل، بمجرد إتمام عملية مصافحة TCP الثلاثية
$worker->onConnect = function(TcpConnection $connection) {
   /**
    * استدعاء onWebSocketConnect عند مصافحة توصيل websocket العميل
    * في onWebSocketConnect، سيتم الحصول على قيمة X_REAL_IP المرسلة من قبل nginx من خلال رأس http
    */
   $connection->onWebSocketConnect = function(TcpConnection $connection){
       /**
        * كائن الاتصال ليس لديه خاصية realIP، لذا نقوم بإضافة خاصية realIP ديناميكيًا إلى كائن الاتصال
        * تذكر أنه يمكن إضافة الخاصية ديناميكيًا إلى كائنات PHP، يمكنك أيضًا استخدام اسم خاصية تفضله
        */
       $connection->realIP = $_SERVER['HTTP_X_REAL_IP'];
   };
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // عند استخدام عنوان IP العميل، يمكنك استخدام $connection->realIP مباشرة
    $connection->send($connection->realIP);
};
Worker::runAll();
```

**للحصول على عنوان IP العميل المُعد من قبل nginx في GatewayWorker:**

يجب إضافة الكود التالي إلى Events.php
```php
class Events
{
   public static function onWebsocketConnect($client_id, $data)
   {    
        $_SESSION['realIP'] = $data['server']['HTTP_X_REAL_IP'];
   }
   // .... الكود الآخر المحذوف....
}
```
بعد إضافة هذا الكود، يجب إعادة تشغيل GatewayWorker.

بهذا الشكل، يمكنك الحصول على عنوان IP العميل الحقيقي في الأحداث `onMessage` و `onClose` في Events.php من خلال `$_SESSION['realIP']`. 

> ملحوظة: لا يمكن استخدام `$_SESSION['realIP']` مباشرة في أحداث `onWorkerStart`، `onConnect`، و `onWorkerStop` داخل Events.php.
