# إنشاء خدمة wss 

**السؤال:**
كيف يمكن إنشاء خدمة wss باستخدام Workerman، حتى يمكن للعميل الاتصال بالخادم باستخدام بروتوكول wss، مثلاً في تطبيق الويب الصغير في WeChat.

**الإجابة:**
بروتوكول wss هو في الواقع [websocket](https://baike.baidu.com/item/WebSocket) + [SSL](https://baike.baidu.com/item/ssl)، أي نوع في إضافة طبقة [SSL](https://baike.baidu.com/item/ssl) إلى بروتوكول websocket، مشابه لـ [https](https://baike.baidu.com/item/https) ([http](https://baike.baidu.com/item/http)+[SSL](https://baike.baidu.com/item/ssl)). لذلك، يمكن دعم بروتوكول wss ببساطة من خلال تفعيل [SSL](https://baike.baidu.com/item/ssl) على أساس بروتوكول [websocket](https://baike.baidu.com/item/WebSocket).

## الطريقة الأولى: استخدام nginx/apache كوكيل لبروتوكول SSL (مستحسن)

**مبدأ الاتصال وسير العمل**

1. العميل يبدأ الاتصال الwss ويتصل بـ nginx/apache.
2. يحول nginx/apache البيانات الwss إلى بيانات الws ثم يعيد توجيهها إلى منفذ بروتوكول websocket في Workerman.
3. بمجرّد استلام Workerman للبيانات، يقوم بمعالجة الخطط التشغيلية.
4. عندما يريد Workerman إرسال رسالة إلى العميل، يتم العكس من هذه العملية، حيث يتم تحويل البيانات عبر nginx/apache إلى بروتوكول wss ثم يتم إرسالها إلى العميل.

## إعداد nginx

**الشروط والاستعدادات:**

1. يجب أن يكون nginx مثبّتاً بإصدار لا يقل عن 1.3.
2. نفترض أن Workerman يستمع على منفذ 8282 (بروتوكول websocket).
3. لقد قمت بطلب الشهادة (ملف pem/crt وملف مفتاح)، ونفترض وضعها في /etc/nginx/conf.d/ssl.
4. تخطط لاستخدام nginx لفتح ميناء 443 لتوفير خدمة وكيل wss إلى العامة (يمكن تعديل الميناء حسب الحاجة).
5. عادةً ما يعمل nginx كخادم مواقع لخدمات أخرى، لتجنب التأثير على استخدام الموقع الأصلي، سيتم استخدام العنوان domain.com/wss كنقطة الوصول لخدمة الوكيل wss. يعني عنوان الاتصال للعميل هو wss://domain.com/wss.

**تكوين nginx مشابه للتالي**:
```nginx
server {
  listen 443;
  # تجاهل تكوين domain...

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
    proxy_set_header X-Real-IP $remote_addr;
  }
  
  # location / {} إعدادات أخرى للموقع...
}
```
**الاختبار**

```javascript
// الشهادة ستتحقق من عنوان الاتصال، الرجاء استخدام عنوان الاتصال(domain) هنا
ws = new WebSocket("wss://domain.com/wss");

ws.onopen = function() {
    alert("تم الاتصال بنجاح");
    ws.send('tom');
    alert("إرسال سلسلة نصية إلى الخادم: tom");
};
ws.onmessage = function(e) {
    alert("استقبلت رسالة من الخادم: " + e.data);
};
```
## استخدام apache كوكيل wss

يمكن أيضاً استخدام apache كوكيل لإعادة توجيه wss إلى Workerman.

**الاستعداد:**

1. استمع GatewayWorker على الميناء 8282 (بروتوكول websocket).
2. لقد طلبت شهادة SSL، ونفترض وضعها في /server/httpd/cert/.
3. استخدم apache لتحويل منفذ 443 إلى المنفذ 8282 المحدد.
4. تم تحميل ملف httpd-ssl.conf.
5. تم تثبيت openssl.

**تمكين proxy_wstunnel_module**
```apache
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_wstunnel_module modules/mod_proxy_wstunnel.so
```
**تكوين SSL والوكيل**
```apache
DocumentRoot "/مسار/الموقع"
ServerName domain

# Proxy Config
SSLProxyEngine on

ProxyRequests Off
ProxyPass /wss ws://127.0.0.1:8282/wss
ProxyPassReverse /wss ws://127.0.0.1:8282/wss

# إضافة دعم بروتوكول SSL، والتخلص من البروتوكولات غير الآمنة
SSLProtocol all -SSLv2 -SSLv3
# تغيير مجموعة التشفير إلى الآتي
SSLCipherSuite HIGH:!RC4:!MD5:!aNULL:!eNULL:!NULL:!DH:!EDH:!EXP:+MEDIUM
SSLHonorCipherOrder on
# تكوين مفتاح شهادة
SSLCertificateFile /server/httpd/cert/your.pem
# تكوين مفتاح خاص شهادة
SSLCertificateKeyFile /server/httpd/cert/your.key
# تكوين سلسلة الشهادة
SSLCertificateChainFile /server/httpd/cert/chain.pem
```

**الاختبار**
```javascript
// الشهادة ستتحقق من عنوان الاتصال، الرجاء استخدام عنوان الاتصال(domain) هنا
ws = new WebSocket("wss://domain/wss");

ws.onopen = function() {
    alert("تم الاتصال بنجاح");
    ws.send('tom');
    alert("إرسال سلسلة نصية إلى الخادم: tom");
};
ws.onmessage = function(e) {
    alert("استقبلت رسالة من الخادم: " + e.data);
};
```

## الطريقة الثانية: استخدام Workerman مباشرة لفتح SSL (غير مستحسن)

> **ملاحظة**
> لا يمكن تفعيل nginx/apache و Workerman في نفس الوقت، يتعين اختيار أحدهما.

**الاستعداد:**

1. Workerman الإصدار >= 3.3.7
2. تثبيت إضافة openssl في PHP
3. لقد طلبت الشهادة ونفترض وضعها في أي مجلد على القرص.

**الكود:**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// الشهادة يفضل أن تكون شهادة معتمدة
$context = array(
    // المزيد من خيارات SSL يرجى مراجعة الدليل http://php.net/manual/zh/context.ssl.php
    'ssl' => array(
        // الرجاء استخدام المسار المطلق
        'local_cert'        => 'مسار القرص/server.pem', // يمكن أن يكون ملف crt
        'local_pk'          => 'مسار القرص/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // إذا كانت الشهادة موقعة ذاتياً، يجب تمكين هذا الخيار
    )
);
// تعيين transport لفتح ssl، وبالتالي websocket+ssl عبارة عن wss
$worker = new Worker('websocket://0.0.0.0:8282', $context);
$worker->transport = 'ssl'; // تعيين نقل البيانات عبر ssl
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

من خلال هذا الكود، أصبح Workerman يسمع بروتوكول wss، والعميل يمكنه الآن الاتصال بـ Workerman لتحقيق الاتصال الآمن عبر الإنترنت.

**الاختبار**

افتح متصفح Chrome، اضغط على F12 لفتح لوحة التحكم في التصحيح، ثم اكتب الأمر التالي في جهاز وحدتك (أو يمكنك نسخ الكود ووضعه في صفحة html وتشغيله إلى ملف):

```javascript
// الشهادة ستتحقق من عنوان الاتصال، الرجاء استخدام عنوان الاتصال(domain) هنا، تنبيه: يجب كتابة رقم الميناء هنا
ws = new WebSocket("wss://domain.com:8282");
ws.onopen = function() {
    alert("تم الاتصال بنجاح");
    ws.send('tom');
    alert("إرسال سلسلة نصية إلى الخادم: tom");
};
ws.onmessage = function(e) {
    alert("استقبلت رسالة من الخادم: " + e.data);
};
```

**ملاحظات:**

1. إذا كان من الضروري استخدام ميناء 443، يفضل استخدام الطريقة الأولى واستخدام nginx/apache لتحقيق wss.
2. ميناء wss يمكن الوصول إليه فقط عبر البروتوكول wss، ويتعذر الوصول إليه عبر ws.
3. عادة ما يتم ربط الشهادة بالنطاق، لذا يجب على العميل استخدام النطاق للاتصال، ويجب تجنب استخدام عنوان IP.
4. في حالة عدم القدرة على الوصول، يرجى التحقق من جدار الحماية للخادم.
5. هذه الطريقة تتطلب إصدار PHP >= 5.6، لأن تطبيقات WeChat تتطلب tls1.2، وإصدارات PHP أقل من 5.6 لا تدعم tls1.2.

مقالات ذات صلة:  
[الحصول على عنوان IP الحقيقي للعميل من خلال الوكيل](get-real-ip-from-proxy.md)  
[مرجع خيارات سياق SSL ل Workerman](https://php.net/manual/zh/context.ssl.php)
