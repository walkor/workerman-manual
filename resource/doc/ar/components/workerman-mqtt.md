MQTT هو بروتوكول نقل الرسائل بنمط النشر / الاشتراك الذي يعتمد على بنية عميل / خادم وقد أصبح جزءًا مهمًا من الإنترنت الأشياء. تصميمه يعتمد على الخفة والانفتاح والبساطة والمواصفات وسهولة التنفيذ. هذه السمات تجعله خيارًا جيدًا للعديد من السيناريوهات ، خاصة بالنسبة للبيئات المحدودة مثل اتصال الجهاز بالجهاز (M2M) وبيئات الإنترنت الأشياء (IoT).

workerman\mqtt هو مكتبة عميل MQTT غير متزامنة تعتمد على workerman ، يمكن استخدامها لاستقبال أو إرسال رسائل بروتوكول MQTT. يدعم "QoS 0" و "QoS 1" و "QoS 2". يدعم الإصدارات "MQTT" "3.1" "3.1.1" "5".

# عنوان المشروع
https://github.com/walkor/mqtt

# التثبيت
```
composer require workerman/mqtt
```

# مثال
**subscribe.php**
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function(){
    $mqtt = new Workerman\Mqtt\Client('mqtt://test.mosquitto.org:1883');
    $mqtt->onConnect = function($mqtt) {
        $mqtt->subscribe('test');
    };
    $mqtt->onMessage = function($topic, $content){
        var_dump($topic, $content);
    };
    $mqtt->connect();
};
Worker::runAll();
```
تشغيل من سطر الأوامر  ```php subscribe.php start``` للتشغيل

**publish.php**
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function(){
    $mqtt = new Workerman\Mqtt\Client('mqtt://test.mosquitto.org:1883');
    $mqtt->onConnect = function($mqtt) {
       $mqtt->publish('test', 'hello workerman mqtt');
    };
    $mqtt->connect();
};
Worker::runAll();
```

تشغيل من سطر الأوامر ```php publish.php start``` للتشغيل.

## واجهة workerman\mqtt\Client

  * <a href="#construct"><code>Client::<b>__construct()</b></code></a>
  * <a href="#connect"><code>Client::<b>connect()</b></code></a>
  * <a href="#publish"><code>Client::<b>publish()</b></code></a>
  * <a href="#subscribe"><code>Client::<b>subscribe()</b></code></a>
  * <a href="#unsubscribe"><code>Client::<b>unsubscribe()</b></code></a>
  * <a href="#disconnect"><code>Client::<b>disconnect()</b></code></a>
  * <a href="#close"><code>Client::<b>close()</b></code></a>
  * <a href="#onConnect"><code>callback <b>onConnect</b></code></a>
  * <a href="#onMessage"><code>callback <b>onMessage</b></code></a>
  * <a href="#onError"><code>callback <b>onError</b></code></a>
  * <a href="#onClose"><code>callback <b>onClose</b></code></a>
