# الأحداث المدعومة حاليًا من خلال workerman

| الاسم | الاعتماديات | دعم التعاقب | الأولوية | إصدار workerman |
| ----- | ------ | - | ----- |
|  Workerman\Events\Select   |   لا يوجد   | غير مدعوم  |  الدعم الافتراضي للنواة   |  >=3.0  ｜
|  Workerman\Events\Revolt   |   event(اختياري)   | مدعوم |  يُحتاج [revolt/event-loop](https://github.com/revoltphp/event-loop)   |  >=5.0  |
|  Workerman\Events\Event   |   event  | غير مدعوم |  الدعم الافتراضي للنواة   |  >=3.0  |
|  Workerman\Events\Swoole   |  [swoole](https://github.com/swoole/swoole-src)   | مدعوم |  يُحتاج ضبط يدوي   |  >=4.0  |
|  Workerman\Events\Swow   |   [swow](https://github.com/swow/swow)   | مدعوم |  يُحتاج ضبط يدوي   |  >=5.0  |

* سيقدم كل محرك نواة ميزات فريدة ، على سبيل المثال ، سيدمج استخدام `Revolt` workerman دعم [Fiber (فيبر) المدمجة في PHP](https://www.php.net/manual/zh/language.fibers.php) ، أما بالنسبة لاستخدام `Swoole` سيضمن دعم البرمجة المتعددة لمصفوفات الـ Swoole
* بين المحركات الحالية ، فهي تتعارض مع بعضها البعض ، على سبيل المثال ، عند استخدام البرمجة الفرعية Fiber في `Revolt` ، لا يمكن استخدام برمجة المصفوفة في `Swoole` أو `Swow`
* يتطلب `Revolt` تثبيت `composer require revolt/event-loop ^1.0.0` ، بعد التثبيت ستقوم النواة الداخلية لـ workerman تلقائيًا بتعيينه كمحرك أحداث مفضل
* يتطلب `Swoole` و `Swow` ضبط يدوي لـ `Worker::$eventLoopClass` ليعملان (يرجى الرجوع إلى الفقرة التالية)
* بشكل افتراضي لا يتم تمكين [تشغيل البرمجة المتعددة الفوري](https://wiki.swoole.com/#/runtime?id=runtime) في Swoole ، مما يعني أن استخدام Pdo وRedis وقراءة وكتابة الملفات المدمجة في PHP لا تزال تتطلب استدعاءًا مُتزامنًا
* في حالة رغبتك في تمكين تشغيل البرمجة المتعددة الفوري في Swoole ، ستحتاج إلى استدعاء `\Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_ALL);` يدويًا

> **ملاحظة**
> سيؤدي تثبيت مُلحق swow تلقائيًا إلى تغيير بعض وظائف PHP المدمجة ، وهذا سيؤدي إلى عدم قدرة workerman على الاستجابة للطلبات والإشارات إن لم يتم استخدام swow كمحرك أحداث. لذا إذا كنت لا تستخدم swow كمحرك أساسي فيجب عليك تعليق swow في ملف php.ini

لمزيد من المعلومات راجع [workerman البرمجة المتعددة](../fiber.md)

# ضبط أحداث workerman

فيما يلي كيفية ضبط أحداث workerman يدويًا

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Swoole\Coroutine\Http\Client;
use Swoole\Coroutine;

// ضبط الحدث الأساسي يدويًا
Worker::$eventLoopClass = Workerman\Events\Revolt::class;
//Worker::$eventLoopClass = Workerman\Events\Select::class;
//Worker::$eventLoopClass = Workerman\Events\Event::class;
//Worker::$eventLoopClass = Workerman\Events\Swoole::class;
//Worker::$eventLoopClass = Workerman\Events\Swow::class;
$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    Coroutine::create(function() use ($connection) {
        $cli = new Client('example.com', 80);
        $cli->get('/get');
        $connection->send($cli->body);
    });
};

Worker::runAll();
```
