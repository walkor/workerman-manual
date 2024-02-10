# وركمان-ريديس

## مقدمة

workeman/redis هو مكون Redis غير متزامن يعتمد على Workerman.

> **ملاحظة**
> الهدف الرئيسي لهذا المشروع هو تحقيق اشتراك غير متزامن في Redis (subscribe، pSubscribe). 
> لأن Redis سريع بما فيه الكفاية، فإنه إذا لم يكن هناك حاجة فعلية للاشتراك الغير متزامن في الـ psubscribe subscribe، فلن يكون هناك حاجة لاستخدام هذا العميل غير المتزامن، قد يوفر استخدام امتداد Redis أداءً أفضل.

## التثبيت:

```composer require workerman/redis```

## الاستخدام بواسطة الاستدعاء العادي

```php
use Workerman\Worker;
use Workerman\Redis\Client;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:6161');

$worker->onWorkerStart = function() {
    global $redis;
    $redis = new Client('redis://127.0.0.1:6379');
};

$worker->onMessage = function(TcpConnection $connection, $data) {
    global $redis;
    $redis->set('key', 'hello world');    
    $redis->get('key', function ($result) use ($connection) {
        $connection->send($result);
    });  
};

Worker::runAll();
```

## الاستخدام بواسطة التنظيم

> **ملاحظة**
> استخدام التنظيم يتطلب workerman >= 5.0، workerman/redis >= 2.0.0 وتثبيت composer require revolt/event-loop ^1.0.0.

```php
use Workerman\Worker;
use Workerman\Redis\Client;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:6161');

$worker->onWorkerStart = function() {
    global $redis;
    $redis = new Client('redis://127.0.0.1:6379');
};

$worker->onMessage = function(TcpConnection $connection, $data) {
    global $redis;
    $redis->set('key', 'hello world');    
    $result = $redis->get('key');
    $connection->send($result);
};

Worker::runAll();
```

عند عدم تعيين وظيفة استدعاء، يقوم العميل بإرجاع نتائج الطلبات الغير متزامنة بشكل متزامن، وعملية الطلب لا تحجب العملية الجارية حالياً، مما يعني أنه يمكن معالجة الطلبات بشكل متوازي.

> **ملاحظة**
> لا يدعم التنظيم استخدام psubscribe subscribe.

# الوثائق
**توضيح**

**في طريقة الاستدعاء الراداري، تكون الدالة الاستدعائية عادةً ما تحتوي على معاملين ($result، $redis)، حيث يكون `$result` هو النتيجة و `$redis` هو نسخة Redis. على سبيل المثال:**
```php
use Workerman\Redis\Client;
$redis = new Client('redis://127.0.0.1:6379');
// تعيين الدالة الاستدعائية لتحديد نتيجة الاستدعاء set
$redis->set('key', 'value', function ($result, $redis) {
    var_dump($result); // true
});
// الدوال الاستدعائية اختيارية، وهنا يتم حذف الدالة الاستدعائية
$redis->set('key1', 'value1');
// يمكن تداخل الدوال الاستدعائية
$redis->get('key', function ($result, $redis){
    $redis->set('key2', 'value2', function ($result) {
        var_dump($result);
    });
});
```

## **الاتصال**
```php
use Workerman\Redis\Client;
// الدوال الاستدعائية مستبينة
$redis = new Client('redis://127.0.0.1:6379');
// مع دالة استدعائية
$redis = new Client('redis://127.0.0.1:6379', [
    'connect_timeout' => 10 // تعيين مهلة الاتصال بـ 10 ثوانٍ، إذا لم يتم التعيين، ستكون القيمة الافتراضية 5 ثوانٍ
], function ($success, $redis) {
    // دالة الاستدعاء الناتجة عن الاتصال
    if (!$success) echo $redis->error();
});
```

## **auth**
```php
// التحقق من كلمة المرور
$redis->auth('password', function ($result) {
    
});
// التحقق من اسم المستخدم وكلمة المرور
$redis->auth('username', 'password', function ($result) {

});
```

## **pSubscribe**

للاشتراك في قناة واحدة أو أكثر تطابق نمط محدد.

يتمثل كل نمط في استخدام \* كرمز مطابقة، على سبيل المثال it\* يطابق كل القنوات التي تبدأ بـ it (مثل it.news  و it.blog و it.tweets وما إلى ذلك). news.\* يطابق كل القنوات التي تبدأ بـ news. (مثل news.it و news.global.today، وما إلى ذلك)، وهكذا.

ملاحظة: تحتوي دالة الاستدعاء pSubscribe على 4 معاملات ($pattern، $channel، $message، $redis).

بمجرد استدعاء النسخة $redis لواجهة pSubscribe أو subscribe، سيتم تجاهل أي استدعاء آخر للنسخة الحالية.

```php
$redis = new Client('redis://127.0.0.1:6379');
$redis2 = new Client('redis://127.0.0.1:6379');
$redis->psubscribe(['news*', 'blog*'], function ($pattern, $channel, $message) {
    echo "$pattern, $channel, $message"; // news*, news.add, news content
});

Timer::add(5, function () use ($redis2){
    $redis2->publish('news.add', 'news content');
});
```

## **subscribe**

يستخدم للاشتراك في معلومات قناة واحدة أو أكثر معينة.

ملاحظة: تحتوي دالة الاستدعاء subscribe على 3 معاملات ($channel، $message، $redis).

بمجرد استدعاء النسخة $redis لواجهة pSubscribe أو subscribe، سيتم تجاهل أي استدعاء آخر للنسخة الحالية.

```php
$redis = new Client('redis://127.0.0.1:6379');
$redis2 = new Client('redis://127.0.0.1:6379');
$redis->subscribe(['news', 'blog'], function ($channel, $message) {
    echo "$channel, $message"; // news, news content
});

Timer::add(5, function () use ($redis2){
    $redis2->publish('news', 'news content');
});
```

## **publish**

يستخدم لإرسال معلومات محددة إلى القناة المعينة.

يعيد عدد المشتركين الذين استقبلوا المعلومات.

```php
$redis2->publish('news', 'news content');
```
## **select**
```php
// تجاهل التعليق
$redis->select(2);
$redis->select('test', function ($result, $redis) {
    // يجب أن يكون الحد الأدنى الذي يتم تحديده للوظيفة select عبارة عن رقم، لذلك سيكون $result هنا false
    var_dump($result, $redis->error());
});
```

## **get**

يُستخدم الأمر للحصول على قيمة مفتاح محدد. إذا كان المفتاح غير موجود، سيُرجع NULL. إذا كانت القيمة المخزنة للمفتاح ليست من نوع السلسلة، سيُرجع false.
```php
$redis->get('key', function ($result) {
     // إذا كان المفتاح غير موجود، سيُرجع NULL، وإذا حدثت خطأ، سيُرجع false
    var_dump($result);
});
```

## **set**

يُستخدم لتعيين قيمة مفتاح معين. إذا كان المفتاح موجودًا بالفعل، سيتم كتابة القيمة الجديدة وتجاوز القيمة القديمة، دون النظر إلى النوع.
```php
$redis->set('key', 'value');
$redis->set('key', 'value', function ($result) {});
// يمكن تمرير الوقت الانتهاء، فعلى سبيل المثال، الانتهاء بعد 10 ثوان
$redis->set('key', 'value', 10);
$redis->set('key', 'value', 10, function ($result) {});
```

## **setEx, pSetEx**

يُستخدم لتعيين قيمة محددة للمفتاح مع وقت انتهاء. إذا كان المفتاح موجودًا بالفعل، سيقوم أمر SETEX بتغيير القيمة القديمة.
```php
// يجب ملاحظة أن الوقت الانتهاء يجب تمريره كمعامل ثاني بوحدة الثانية
$redis->setEx('key', 3600, 'value'); 
// وحدة الوقت لـ pSetEx هي الوحدة بالميلي ثانية
$redis->pSetEx('key', 3600, 'value'); 
```

## **del**

يُستخدم لحذف مفتاح موجود. يُرجع النتيجة على شكل رقم يُمثل كم عدد المفاتيح التي تم حذفها(المفاتيح الغير موجودة لا تُعتبر).
```php
// لحذف مفتاح واحد
$redis->del('key');
// لحذف مفاتيح متعددة
$redis->del(['key', 'key1', 'key2']);
```

## **setNx**

(SET if Not eXists) يُستخدم عندما لا يكون المفتاح موجودًا، لتعيين قيمة معينة للمفتاح.
```php
$redis->del('key');
$redis->setNx('key', 'value', function ($result) {
    var_dump($result); // 1
});
$redis->setNx('key', 'value', function ($result) {
    var_dump($result); // 0
});
```

## **exists**

يُستخدم للتحقق ما إذا كان المفتاح المعطى موجودًا. يُرجع النتيجة على شكل رقم يُمثل عدد المفاتيح الموجودة.
```php
$redis->set('key', 'value');
$redis->exists('key', function ($result) {
    var_dump($result); // 1
}); 
$redis->exists('NonExistingKey', function ($result) {
    var_dump($result); // 0
}); 

$redis->mset(['foo' => 'foo', 'bar' => 'bar', 'baz' => 'baz']);
$redis->exists(['foo', 'bar', 'baz'], function ($result) {
    var_dump($result); // 3
}); 
```

## **incr, incrBy**

يُستخدم لزيادة قيمة المفتاح بمقدار واحد/مقدار محدد. إذا كان المفتاح غير موجود، سيتم تهيئة قيمة المفتاح على أنها 0، ثم يُنفذ الأمر incr/incrBy.
إذا كانت القيمة ذات النوع غير الصحيح، أو لا يُمكن تمثيل القيمة النصية كرقم، سيُرجع false.
في حال النجاح سيتم إرجاع القيمة المضافة بعد الزيادة.
```php
$redis->incr('key1', function ($result) {
    var_dump($result);
}); 
$redis->incrBy('key1', 10, function ($result) {
    var_dump($result);
}); 
```

## **incrByFloat**

يُستخدم لزيادة قيمة مفتاح معين بمقدار عدد كسري معين.
إذا كان المفتاح غير موجود، سيتم تهيئة قيمة المفتاح على أنها 0، ثم سيتم تنفيذ عملية الجمع incrByFloat.
إذا كانت القيمة ذات النوع غير الصحيح، أو لا يُمكن تمثيل القيمة النصية كرقم، سيُرجع false.
في حال النجاح سيتم إرجاع القيمة المضافة بعد الزيادة.
```php
$redis->incrByFloat('key1', 1.5, function ($result) {
    var_dump($result);
}); 
```

## **decr, decrBy**

يُستخدم للنقص قيمة المفتاح بمقدار واحد/مقدار محدد.
إذا كان المفتاح غير موجود، سيتم تهيئة قيمة المفتاح على أنها 0، ثم يُنفذ الأمر decr/decrBy.
إذا كانت القيمة ذات النوع غير الصحيح، أو لا يُمكن تمثيل القيمة النصية كرقم، سيُرجع false.
في حال النجاح سيتم إرجاع القيمة المنقوصة بعد النقص.
```php
$redis->decr('key1', function ($result) {
    var_dump($result);
}); 
$redis->decrBy('key1', 10, function ($result) {
    var_dump($result);
}); 
```

## **mGet**

يُرجع قيمة كل(مفتاح واحد أو أكثر) المفاتيح المعطاة. إذا كان أحد المفاتيح غير موجود، سيُرجع NULL.
```php
$redis->set('key1', 'value1');
$redis->set('key2', 'value2');
$redis->set('key3', 'value3');
$redis->mGet(['key0', 'key1', 'key5'], function ($result) {
    var_dump($result); // [null, 'value1', null];
}); 
```

## **getSet**

يُستخدم لتعيين قيمة مفتاح محدد وإرجاع القيمة القديمة للمفتاح.
```php
$redis->set('x', '42');
$redis->getSet('x', 'lol', function ($result) {
    var_dump($result); // '42'
}) ;
$redis->get('x', function ($result) {
    var_dump($result); // 'lol'
}) ;
```

## **randomKey**

يُرجع مفتاح عشوائي من قاعدة البيانات الحالية.
```php
$redis->randomKey(function ($key) use ($redis) {
    $redis->get($key, function ($result) {
        var_dump($result); 
    }) ;
})
```

## **move**

يُنقل مفتاح قاعدة البيانات الحالية إلى قاعدة البيانات المعطاة.
```php
$redis->select(0);	// التبديل إلى قاعدة البيانات 0
$redis->set('x', '42');	// كتابة 42 إلى x
$redis->move('x', 1, function ($result) { 	// نقلها إلى قاعدة البيانات 1
    var_dump($result); // 1
}) ;  
$redis->select(1);	// التبديل إلى قاعدة البيانات 1
$redis->get('x', function ($result) {
    var_dump($result); // '42'
}) ;
```
## **إعادة التسمية**

تغيير اسم المفتاح، سيعيد `false` إذا كان المفتاح غير موجود.
```php
$redis->set('x', '42');
$redis->rename('x', 'y', function ($result) {
    var_dump($result); // true
}) ;
```

## **إعادة التسمية بدون احتساب الإضافة**

عندما يكون المفتاح الجديد غير موجود، يتم تغيير اسم المفتاح.
```php
$redis->del('y');
$redis->set('x', '42');
$redis->renameNx('x', 'y', function ($result) {
    var_dump($result); // 1
}) ;
```

## **تنتهي الصلاحية**

تعيين وقت انتهاء صلاحية المفتاح بحيث بعد انتهاء صلاحيته، لن يكون المفتاح متاحًا بعد الآن. الوحدة بالثواني. إذا نجحت ستعيد 1، إذا كان المفتاح غير موجود سترجع 0، وفي حالة حدوث خطأ سترجع `false`.
```php
$redis->set('x', '42');
$redis->expire('x', 3);
```

## **المفاتيح**

تستخدم هذه الأمر للبحث عن جميع المفاتيح المطابقة لنمط المفتاح المحدد.
```php
$redis->keys('*', function ($keys) {
    var_dump($keys); 
}) ;
$redis->keys('user*', function ($keys) {
    var_dump($keys); 
}) ;
```

## **النوع**

تعيد نوع القيمة المخزنة في المفتاح. يمكن أن تكون النتيجة إما "string" أو "set" أو "list" أو "zset" أو "hash"، حيث تعني "none" عدم وجود المفتاح.
```php
$redis->type('key', function ($result) {
    var_dump($result); // string set list zset hash none
}) ;
```

## **إلحاق النص**

إذا كان المفتاح موجودًا بالفعل وكان نصًا، يقوم أمر APPEND بإلحاق القيمة بنهاية المفتاح الحالي ويعيد طول النص الجديد.

في حالة عدم وجود المفتاح، يقوم APPEND ببساطة تعيين المفتاح المعطى إليه القيمة المحددة ويعيد طول النص. 

إذا كان المفتاح موجودًا ولكنه ليس نصًا، سترجع `false`.

```php
$redis->set('key', 'value1');
$redis->append('key', 'value2', function ($result) {
    var_dump($result); // 12
}) ; 
$redis->get('key', function ($result) {
    var_dump($result); // 'value1value2'
}) ;
```

## **الحصول على النطاق**

الحصول على مجموعة فرعية من النص المخزن في المفتاح المحدد. يتم تحديد نطاق الاقتطاع بواسطة نقطة البداية ونقطة النهاية (بما في ذلك نقطة البداية ونقطة النهاية). إذا لم يكن المفتاح موجودًا، ستعود سلسلة فارغة. إذا كان المفتاح ليس من نوع النص، ستعود `false`.
```php
$redis->set('key', 'string value');
$redis->getRange('key', 0, 5, function ($result) {
    var_dump($result); // 'string'
}) ; 
$redis->getRange('key', -5, -1 , function ($result) {
    var_dump($result); // 'value'
}) ;
```

## **تعيين النطاق**

يغطي المقدمة بناء على القيمة النصية المحددة في المفتاح المطلوب، بدءًا من موقع الإزاحة. إذا كان المفتاح غير موجود، فسيتم ضبطه على النص المحدد. في حالة عدم كون المفتاح من نوع النص، ستعود `false`.

تعيد النتيجة طول النص الجديد.
```php
$redis->set('key', 'Hello world');
$redis->setRange('key', 6, "redis", function ($result) {
    var_dump($result); // 11
}) ; 
$redis->get('key', function ($result) {
    var_dump($result); // 'Hello redis'
}) ; 
```

## **طول النص**

الحصول على طول قيمة النص المحددة في المفتاح المقدم. عندما لا يحتوي المفتاح على قيمة نصية، ستعود `false`.
```php
$redis->set('key', 'value');
$redis->strlen('key', function ($result) {
    var_dump($result); // 5
}) ; 
```

## **الحصول على البت**

للحصول على البت (البتات) المحددة في المفتاح المحدد.
```php
$redis->set('key', "\x7f"); // this is 0111 1111
$redis->getBit('key', 0, function ($result) {
    var_dump($result); // 0
}) ; 
```

## **تعيين البت**

لتعيين أو مسح البت (البتات) المحددة في المفتاح المحدد. تعيد القيمة 0 أو 1، وهي القيمة قبل التعديل.
```php
$redis->set('key', "*");	// ord("*") = 42 = 0x2a = "0010 1010"
$redis->setBit('key', 5, 1, function ($result) {
    var_dump($result); // 0
}) ; 
```

## **عملية البت**

تنفيذ عملية البت على مفاتيح متعددة (تحتوي على قيم نصية) وتخزين النتيجة في المفتاح الهدف.

تدعم أمر BITOP أربع عمليات بتية: **AND**، **OR**، **XOR**، و**NOT**.

تعيد النتيجة طول النص الذي يتم تخزينه في المفتاح الهدف، وهو يساوي أقصى طول للسلاسل المدخلة.
```php
$redis->set('key1', "abc");
$redis->bitOp( 'AND', 'dst', 'key1', 'key2', function ($result) {
    var_dump($result); // 3
}) ;
```

## **عد البتات**

حساب عدد البتات المضبوطة في النص (احتساب السكان).

بشكل افتراضي، يتم فحص جميع بايتات النص. يمكن تحديد نطاق العملية الحسابية بتحويلها إلى المعلمات الإضافية *start* و *end*.

بالمثل GETRANGE أمر، يمكن أن تتضمن البداية والنهاية قيمًا سالبةً، بحيث تبدأ في ترقيم بايت السلسلة من النهاية، حيث أن -1 هو البايت الأخير، و -2 هو البايت قبل الأخير، وهكذا.

النتيجة تعيد عدد البتات المُضبطة بقيمة 1 في النص. يُعَد المفتاح الغير موجود نفسه نصًا فارغًا، وبالتالي سيُرجِع الأمر الناتج صفر.
```php
$redis->set('key', 'hello');
$redis->bitCount( 'key', 0, 0, function ($result) {
    var_dump($result); // 3
}) ;
$redis->bitCount( 'key', function ($result) {
    var_dump($result); //21
}) ;
```
## **الفرز**

يمكن لأمر الفرز (sort) فرز عناصر القائمة (list) أو المجموعة (set) أو المجموعة المرتبة (sorted set).

النموذج: `sort($key, $options, $callback);`

حيث الخيارات (options) هي مفاتيح وقيم اختيارية مثل التالي:

```php
$options = [
    'by' => 'some_pattern_*',
    'limit' => [0, 1],
    'get' => 'some_other_pattern_*', // or an array of patterns
    'sort' => 'asc', // or 'desc'
    'alpha' => true,
    'store' => 'external-key'
];
```

```php
$redis->del('s');
$redis->sAdd('s', 5);
$redis->sAdd('s', 4);
$redis->sAdd('s', 2);
$redis->sAdd('s', 1);
$redis->sAdd('s', 3);
$redis->sort('s', [], function ($result) {
    var_dump($result); // 1,2,3,4,5
}); 
$redis->sort('s', ['sort' => 'desc'], function ($result) {
    var_dump($result); // 5,4,3,2,1
}); 
$redis->sort('s', ['sort' => 'desc', 'store' => 'out'], function ($result) {
    var_dump($result); // (int)5
}); 
```

## **ttl, pttl**

تُرجع الدالة قيمة متبقية لانتهاء صلاحية المفتاح (key) بالثواني / الأمثلة.

إذا لم يكن للمفتاح صلاحية، فإنها تُرجع -1. وإذا كان المفتاح غير موجود، فإنها تُرجع -2.

```php
$redis->set('key', 'value', 10);
// بالثواني
$redis->ttl('key', function ($result) {
    var_dump($result); // 10
});
// بالأمثلة
$redis->pttl('key', function ($result) {
    var_dump($result); // 9999
});
// المفتاح غير موجود
$redis->pttl('key-not-exists', function ($result) {
    var_dump($result); // -2
});
```

## **persist**

تُزيل صلاحية المفتاح المُعطى، مما يجعل المفتاح لا ينتهي أبدًا.

إذا تمت العملية بنجاح، فإنها تُرجع 1. وإذا كان المفتاح غير موجود أو ليس له صلاحية، فإنها تُرجع 0. وإذا حدثت خطأ، فإنها تُرجع false.

```php
$redis->persist('key');
```

## **mSet, mSetNx**

تقوم بضبط مجموعة من الأزواج المفتاح / القيم في عملية واحدة غير قابلة للتجزئة. تُرجع الدالة mSetNx قيمة 1 فقط إذا تم ضبط كل المفاتيح.

تُرجع قيمة 1 في حالة النجاح، و0 في حالة الفشل، وfalse في حالة حدوث خطأ.

```php
$redis->mSet(['key0' => 'value0', 'key1' => 'value1']);
```

## **hSet**

تُعين قيمة لحقل في الجدول الهاشي.

إذا كان الحقل جديداً في الجدول الهاشي وتم ضبط القيمة بنجاح، يتم إرجاع 1. وإذا كان الحقل موجودًا بالفعل في الجدول الهاشي وتم استبدال القيمة القديمة بالقيمة الجديدة، يتم إرجاع 0.

```php
$redis->del('h');
$redis->hSet('h', 'key1', 'hello', function ($r) {
    var_dump($r); // 1
}); 
$redis->hGet('h', 'key1', function ($r) {
    var_dump($r); // hello
}); 
$redis->hSet('h', 'key1', 'plop', function ($r) {
    var_dump($r); // 0
});
$redis->hGet('h', 'key1', function ($r) {
    var_dump($r); // plop
});
```

## **hSetNx**

تقوم بضبط قيمة حقل في الجدول الهاشي إذا كان الحقل غير موجود.

إذا لم يكن الجدول الهاشي موجودًا، سيتم إنشاء جدول هاشي جديد وتنفيذ عملية hSet. وإذا كان الحقل موجودًا بالفعل في الجدول الهاشي، ستكون العملية غير صالحة. وإذا كان المفتاح غير موجود، سيتم إنشاء جدول هاشي جديد وتُنفذ عملية hSetNx.

```php
$redis->del('h');
$redis->hSetNx('h', 'key1', 'hello', function ($r) {
    var_dump($r); // 1
});
$redis->hSetNx('h', 'key1', 'world', function ($r) {
    var_dump($r); // 0
});
```

## **hGet**

تُرجع قيمة الحقل المُحدد في الجدول الهاشي.

إذا كان الحقل المعطى أو المفتاح غير موجود، ستُرجع قيمة null.

```php
$redis->hGet('h', 'key1', function ($result) {
    var_dump($result);
});
```

## **hLen**

مستخدمة للحصول على عدد الحقول في الجدول الهاشي.

عندما يكون المفتاح غير موجود، يُرجع 0.

```php
$redis->del('h');
$redis->hSet('h', 'key1', 'hello');
$redis->hSet('h', 'key2', 'plop');
$redis->hLen('h', function ($result) {
    var_dump($result); // 2
});
```

## **hDel**

تُستخدم لحذف حقل أو أكثر من الحقول في جدول الهاشي، وتتجاهل الحقول التي لا تُوجد.

تُرجع عدد الحقول التي تم حذفها بنجاح، دون حساب الحقول التي تم تجاهلها. وإذا كان المفتاح ليس هاش، ستُرجع false.

```php
$redis->hDel('h', 'key1');
```

## **hKeys**

تُرجع قائمة بكل الحقول في جدول الهاشي.

إذا لم يكن المفتاح موجودًا، ستُرجع قائمة فارغة. وإذا كان المفتاح ليس هاش، ستُرجع false.

```php
$redis->hKeys('key', function ($result) {
    var_dump($result);
});
```

## **hVals**

تُرجع قائمة بقيم جميع الحقول في جدول الهاشي.

إذا لم يكن المفتاح موجودًا، ستُرجع قائمة فارغة. وإذا كان المفتاح ليس هاش، ستُرجع false.

```php
$redis->hVals('key', function ($result) {
    var_dump($result);
});
```

## **hGetAll**

تُرجع قائمة زوجية تحتوي على جميع الحقول وقيم جدول الهاشي.

إذا لم يكن المفتاح موجودًا، ستُرجع مصفوفة فارغة. وإذا كان المفتاح ليس هاش، ستُرجع false.

```php
$redis->del('h');
$redis->hSet('h', 'a', 'x');
$redis->hSet('h', 'b', 'y');
$redis->hSet('h', 'c', 'z');
$redis->hSet('h', 'd', 't');
$redis->hGetAll('h', function ($result) {
    var_export($result); 
});
```
النتيجة:
```php
array (
    'a' => 'x',
    'b' => 'y',
    'c' => 'z',
    'd' => 't',
)
```

## **hExists**

تستخدم للتأكد مما إذا كان حقل محدد موجود في جدول الهاشي أم لا. إذا كان موجودًا، ستُرجع 1. إذا كان الحقل غير موجود أو المفتاح غير موجود، ستُرجع 0. وإذا حدث خطأ، ستُرجع false.

```php
$redis->hExists('h', 'a', function ($result) {
    var_dump($result); //
});
```
## **hIncrBy**

يستخدم لإضافة قيمة محددة إلى حقل في الجدول المجزأة.
 
الزيادة يمكن أن تكون أيضاً بقيمة سالبة، مما يعادل عملية طرح للحقل المحدد.

إذا كانت مفتاح الجدول المجزأة غير موجود، يتم إنشاء جدول مجزأة جديد ويتم تنفيذ أمر HINCRBY.

إذا كان الحقل المحدد غير موجود، فإن قيمة الحقل تكون مبتدئياً بالقيمة 0 قبل تنفيذ الأمر.

سيعيد الأمر HINCRBY قيمة خاطئة إذا تم تخزين قيمة نصية في الحقل.

يتم تحديد القيمة في هذا العملية ضمن حدود التمثيل المؤشر للأرقام الموقعة بـ 64 بت.

```php
$redis->del('h');
$redis->hIncrBy('h', 'x', 2,  function ($result) {
    var_dump($result);
});
```

## **hIncrByFloat**

يشبه الأمر hIncrBy، باستثناء أن الزيادة هي قيمة عائمة.

## **hMSet**

يقوم بتعيين أزواج الحقل-القيمة المتعددة معاً إلى الجدول المجزأة.

سيؤدي هذا الأمر إلى تجاوز الحقول الموجودة بالفعل في جدول المجزأة.

إذا لم يكن جدول المجزأة موجودً، سيتم إنشاء جدول مجزأة فارغ وسيتم تنفيذ عملية HMSET.

```php
$redis->del('h');
$redis->hMSet('h', ['name' => 'Joe', 'sex' => 1])
```

## **hMGet**

يُرجع قيمة أحد أو أكثر من الحقول المحددة في جدول المجزأة بشكل مصفوفة متعلقة.

إذا لم يكن الحقل المحدد موجودًا في جدول المجزأة، سيكون الحقل المقابل له قيمة null. إذا كان المفتاح ليس نوع جدول مجزأة، سيُرجع قيمة خاطئة.

```php
$redis->del('h');
$redis->hSet('h', 'field1', 'value1');
$redis->hSet('h', 'field2', 'value2');
$redis->hMGet('h', ['field1', 'field2', 'field3'], function ($r) {
    var_export($r);
});
```
المخرج
```php
array (
 'field1' => 'value1',
 'field2' => 'value2',
 'field3' => null
)
```

## **blPop, brPop**

يُزيل ويُُرجع العنصر الأول/الأخير في القائمة، وإذا كانت القائمة فارغة فسيتم حجبها حتى يتم انتظار المهلة المحددة أو حتى يتم اكتشاف عنصر يمكن استخراجه.

```php
$redis = new Client('redis://127.0.0.1:6379');
$redis2 = new Client('redis://127.0.0.1:6379');

$redis->blPop(['key1', 'key2'], 10, function ($r) {
    var_export($r); // array ( 0 => 'key1',1 => 'a')
});

Timer::add(1, function () use ($redis2) {
    $redis2->lpush('key1', 'a');
});
```

## **bRPopLPush**

يأخذ العنصر الأخير من القائمة ويدرجه في بداية قائمة أخرى. إذا كانت القائمة فارغة فسيتم حجبها حتى يتم انتظار المهلة المحددة أو حتى يتم اكتشاف عنصر يمكن استخراجه. إذا تجاوزت المهلة فسيتم إرجاع قيمة فارغة.

```php
$redis = new Client('redis://127.0.0.1:6379');
$redis2 = new Client('redis://127.0.0.1:6379');
$redis->del(['key1', 'key2']);
$redis->bRPopLPush('key1', 'key2', 2, function ($r) {
    var_export($r);
});
Timer::add(2, function () use ($redis2) {
    $redis2->lpush('key1', 'a');
    $redis2->lRange('key2', 0, -1, function ($r) {
        var_dump($r);
    });
}, null, false);
```

## **lIndex**

يستخدم للحصول على عنصر معين في القائمة بواسطة فهرس. يمكنك أيضًا استخدام فهارس سالبة، حيث -1 يمثل آخر عنصر في القائمة، و -2 يمثل العنصر قبل الأخير، وهكذا.

إذا كان الفهرس المحدد خارج نطاق القائمة، سيُرجع قيمة null. إذا كان المفتاح المحدد ليس نوع قائمة، سيُرجع قيمة خاطئة.

```php
$redis->del('key1']);
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->lindex('key1', 0, function ($r) {
    var_dump($r); // A
});
```

## **lInsert**

يقوم بإدراج عنصر قبل أو بعد العنصر في القائمة. عندما لا يكون العنصر المحدد موجودًا في القائمة، لا يتم تنفيذ أي عملية.

عندما لا تكون القائمة موجودة، ستُعتبر كقائمة فارغة ولن تتم أي عملية.

إذا لم يكن المفتاح نوع قائمة، سيُرجع قيمة خاطئة.

```php
$redis->del('key1');
$redis->lInsert('key1', 'after', 'A', 'X', function ($r) {
    var_dump($r); // 0
});
$redis->lPush('key1', 'A');
$redis->lPush('key1', 'B');
$redis->lPush('key1', 'C');
$redis->lInsert('key1', 'before', 'C', 'X', function ($r) {
    var_dump($r); // 4
});
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['A', 'B', 'X', 'C']
});
```

## **lPop**

يُزيل ويُعيد العنصر الأول في القائمة.

عندما لا يكون المفتاح موجودًا، ستُرجع قيمة null.

```php
$redis->del('key1');
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->lPop('key1', function ($r) {
    var_dump($r); // A
});
```

## **lPush**

يقوم بإدراج قيمة واحدة أو أكثر في بداية القائمة. إذا لم يكن المفتاح موجودًا، سيتم إنشاء قائمة فارغة وسيتم تنفيذ عملية LPUSH. عندما يكون المفتاح موجودًا وليس من نوع قائمة، سيُرجع قيمة خاطئة.

**ملحوظة:** في Redis قبل الإصدار 2.4 من أمر LPUSH، كان يقبل فقط قيمة واحدة.

```php
$redis->del('key1');
$redis->lPush('key1', 'A');
$redis->lPush('key1', ['B','C']);
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['C', 'B', 'A']
});
```

## **lPushx**

يقوم بإدراج قيمة في بداية القائمة الموجودة، وإذا لم تكن القائمة موجودة فسيكون الأمر غير فعال وسيُرجع 0. إذا كان المفتاح ليس نوع قائمة، سيُرجع قيمة خاطئة.

يُرجع قيمة خلال هذا الأمر هي طول القائمة بعد تنفيذ الأمر lPushx.

```php
$redis->del('key1');
    $redis->lPush('key1', 'A');
$redis->lPushx('key1', ['B','C'], function ($r) {
    var_dump($r); // 3
});
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['C', 'B', 'A']
});
```
## **lRange**

يُرجع عناصر المدى المحدد في القائمة ، حيث يُحدد المدى بالمؤشرات START و END. حيث 0 يمثل العنصر الأول في القائمة ، و 1 يمثل العنصر الثاني في القائمة ، وهكذا. يمكنك أيضًا استخدام فهرس الأرقام السالبة ، حيث يُمثل -1 العنصر الأخير في القائمة ، و -2 يمثل العنصر قبل الأخير في القائمة ، وهكذا.

يُرجع مصفوفة تحتوي على العناصر المحددة في المدى المحدد. إذا لم يكن المفتاح نوعًا قائمة ، فسيتم إرجاع قيمة false.

```php
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->rPush('key1', 'C');
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['C', 'B', 'A']
});
```

## **lRem**

بناءً على قيمة معلم COUNT ، يقوم بإزالة العناصر المتساوية مع القيمة المحددة من القائمة.

قيمة COUNT يمكن أن تكون إحدى القيم التالية:

* count > 0 : بدءًا من رأس الجدول وحتى نهايته ، يزيل عدد COUNT من العناصر المتساوية مع القيمة.
* count < 0 : بدءًا من نهاية الجدول وحتى رأسه ، يزيل عدد COUNT من العناصر المتساوية مع القيمة.
* count = 0 : يقوم بإزالة جميع القيم المتساوية مع القيمة.

يُرجع عدد العناصر التي تمت إزالتها. إذا لم تكن القائمة موجودة ، يتم إرجاع 0. إذا لم تكن القائمة ، يتم إرجاع قيمة false.

```php
$redis->lRem('key1', 2, 'A', function ($r) {
    var_dump($r); 
});
```

## **lSet**

يُستخدم لتعيين قيمة العنصر باستخدام فهرس.

يُرجع true في حالة النجاح ، و false عندما يكون مؤشر الفهرس خارج النطاق أو عندما يتم تطبيق LSET على قائمة فارغة.

```php
$redis->lSet('key1', 0, 'X');
```

## **lTrim**

يُقوم بتقليم القائمة للحفاظ فقط على العناصر في المدى المحدد ، ويزيل العناصر الأخرى.

الفهرس 0 يمثل العنصر الأول في القائمة ، و 1 يمثل العنصر الثاني في القائمة ، وهكذا. يمكنك أيضًا استخدام فهرس الأرقام السالبة ، حيث يُمثل -1 العنصر الأخير في القائمة ، و -2 يمثل العنصر قبل الأخير في القائمة ، وهكذا.

يُرجع true في حالة النجاح ، و false في حالة الفشل.

```php
$redis->del('key1');

$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->rPush('key1', 'C');
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['A', 'B', 'C']
});
$redis->lTrim('key1', 0, 1);
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['A', 'B']
});
```

## **rPop**

يُستخدم لإزالة آخر عنصر من القائمة ويُرجع القيمة التي تمت إزالتها.

عندما لا تكون القائمة موجودة ، يُرجع قيمة null.

```php
$redis->rPop('key1', function ($r) {
    var_dump($r);
});
```

## **rPopLPush**

يُستخدم لإزالة آخر عنصر من القائمة وإضافته إلى قائمة أخرى ومن ثم يُرجع.

```php
$redis->del('x', 'y');
$redis->lPush('x', 'abc');
$redis->lPush('x', 'def');
$redis->lPush('y', '123');
$redis->lPush('y', '456');
$redis->rPopLPush('x', 'y', function ($r) {
    var_dump($r); // abc
});
$redis->lRange('x', 0, -1, function ($r) {
    var_dump($r); // ['def']
});
$redis->lRange('y', 0, -1, function ($r) {
    var_dump($r); // ['abc', '456', '123']
});
```

## **rPush**

يُستخدم لإدراج قيمة واحدة أو أكثر إلى نهاية القائمة (اليمين) ويُرجع طول القائمة بعد الإدراج.

إذا لم تكن القائمة موجودة ، سيتم إنشاء قائمة فارغة ويتم تنفيذ عملية RPUSH. عندما تكون القائمة موجودة ولكنها ليست من نوع قائمة ، سيتم إرجاع قيمة false.

**ملاحظة:** في إصدار Redis 2.4 أو قبل ذلك ، كانت أوامر RPUSH تقبل فقط قيمة واحدة.

```php
$redis->del('key1');
$redis->rPush('key1', 'A', function ($r) {
    var_dump($r); // 1
});
```

## **rPushX**

يُستخدم لإدراج قيمة إلى نهاية القائمة الموجودة بالفعل (اليمين) ويُرجع طول القائمة بعد الإدراج. إذا لم تكن القائمة موجودة ، فإن العملية غير صالحة وسيتم إرجاع 0. عندما تكون القائمة موجودة ولكنها ليست من نوع قائمة ، سيتم إرجاع قيمة false.

```php
$redis->del('key1');
$redis->rPushX('key1', 'A', function ($r) {
    var_dump($r); // 0
});
```

## **lLen**

يُرجع طول القائمة. إذا لم تكن القائمة موجودة ، يُعتبر القائمة فارغة ويتم إرجاع 0. إذا لم تكن القائمة من نوع القائمة ، سيتم إرجاع قيمة false.

```php
$redis->del('key1');
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->rPush('key1', 'C');
$redis->lLen('key1', function ($r) {
    var_dump($r); // 3
});
```

## **sAdd**

يُستخدم لإضافة عنصر واحد أو أكثر إلى المجموعة ، حيث يتم تجاهل العناصر الموجودة بالفعل في المجموعة.

إذا كانت المجموعة key غير موجودة ، سيتم إنشاء مجموعة تحتوي فقط على العناصر المضافة كأعضاء.

عندما لا تكون المجموعة من نوع مجموعة ، سيتم إرجاع قيمة false.

**ملاحظة:** في إصدار Redis 2.4 أو قبل ذلك ، تقبل SADD قيمة عضو واحدة فقط.

```php
$redis->del('key1');
$redis->sAdd('key1' , 'member1');
$redis->sAdd('key1' , ['member2', 'member3'], function ($r) {
    var_dump($r); // 2
});
$redis->sAdd('key1' , 'member2', function ($r) {
    var_dump($r); // 0
});
```
**sCard**

استرجاع عدد العناصر في المجموعة. عندما لا يكون المفتاح موجودًا ، يتم إرجاع 0.

## **sDiff**

استرجاع الفارق بين المجموعة الأولى والمجموعات الأخرى ، يمكن اعتبارها عناصر فريدة في المجموعة الأولى. سيتم اعتبار المفتاح الذي لا يوجد كمجموعة فارغة.

## **sDiffStore**

تخزين فرق المجموعات المعطاة في المجموعة المحددة. إذا كان المفتاح المحدد موجودًا بالفعل ، فسيتم استبداله.

## **sInter**

استرجاع تقاطع كل المجموعات المعطاة. سيتم اعتبار المفتاح الذي لا يوجد كمجموعة فارغة. عندما تحتوي أي مجموعة معطاة على مجموعة فارغة ، ستكون النتيجة أيضًا مجموعة فارغة.

## **sInterStore**

تخزين تقاطع المجموعات المعطاة في المجموعة المحددة واسترجاع عدد عناصر المجموعة التي تم تخزين تقاطعها. إذا كانت المجموعة المحددة موجودة بالفعل ، سيتم استبدالها.

## **sIsMember**

التحقق مما إذا كان عنصر العضو هو عضو في المجموعة. إذا كان عنصر العضو عضوًا في المجموعة ، يتم إرجاع 1. إذا كان عنصر العضو ليس عضوًا في المجموعة ، أو إذا لم يكن المفتاح موجودًا ، فيتم إرجاع 0. إذا كان المفتاح لا يشير إلى نوع مجموعة ، سيتم إرجاع false.

## **sMembers**

استرجاع كافة أعضاء المجموعة. سيتم اعتبار المفتاح الذي لا يوجد كمجموعة فارغة.

## **sMove**

يقوم بنقل عنصر العضو المحدد من مجموعة المصدر إلى مجموعة الوجهة. SMOVE هي عملية ذرية. إذا لم تكن مجموعة المصدر موجودة أو لا تحتوي على عنصر العضو المحدد ، فإن أمر SMOVE لا يقوم بأي عملية ، ويعيد فقط 0. وإلا ، سيتم إزالة عنصر العضو من مجموعة المصدر وإضافته إلى مجموعة الوجهة. عندما تحتوي مجموعة الوجهة بالفعل على عنصر العضو ، يقوم أمر SMOVE ببساطة بحذف عنصر العضو من مجموعة المصدر.

## **sPop**

إزالة عنصر أو عناصر عشوائية من المجموعة المحددة ، ثم يتم إرجاع العناصر المحذوفة. عندما لا تكون المجموعة موجودة أو تكون فارغة ، سيتم إرجاع null.


## **sRandMember**

أمر Redis Srandmember يستخدم لإرجاع عنصر عشوائي من المجموعة.

ابتداءً من إصدار Redis 2.6 ، يقبل أمر Srandmember معلمة count اختيارية:

- إذا كانت count عددًا موجبًا وأقل من قاعدة المجموعة ، فيعيد الأمر مصفوفة تحتوي على عدد count من العناصر ، حيث لا تتكرر العناصر. إذا كانت count أكبر من أو مساوية لقاعدة المجموعة ، فيتم إعادة المجموعة بأكملها.
- إذا كانت count عددًا سالبًا ، فيعيد الأمر مصفوفة حيث قد تتكرر العناصر عدة مرات ، وطول المصفوفة يساوي قيمة count بشكل مطلق.

هذا الأمر مشابه لـ SPOP ، ولكن SPOP يقوم بإزالة العنصر العشوائي من المجموعة ويقوم بإرجاعه، بينما يقوم Srandmember بإرجاع العنصر العشوائي فقط دون أي تغيير على المجموعة.

```php
$redis->del('key1');
$redis->sAdd('key1' , 'member1');
$redis->sAdd('key1' , 'member2');
$redis->sAdd('key1' , 'member3'); 

$redis->sRandMember('key1', function ($r) {
    var_dump($r); // member1
});

$redis->sRandMember('key1', 2, function ($r) {
    var_dump($r); // ['member1', 'member2']
});
$redis->sRandMember('key1', -100, function ($r) {
    var_dump($r); // ['member1', 'member2', 'member3', 'member3', ...]
});
$redis->sRandMember('empty-set', 100, function ($r) {
    var_dump($r); // []
}); 
$redis->sRandMember('not-a-set', 100, function ($r) {
    var_dump($r); // []
});
```

## **sRem**

يزيل عنصرًا واحدًا أو أكثر من أعضاء المجموعة ، ويتجاهل الأعضاء غير الموجودة. يعيد عدد العناصر التي تمت إزالتها بنجاح ، دون احتساب العناصر المتجاهلة. عندما لا يكون key من نوع مجموعة ، يُرجع القيمة false.

في إصدار Redis 2.4 وما قبله ، كان يتم استقبال أمر SREM قيمة عضو واحدة فقط.

```php
$redis->sRem('key1', ['member2', 'member3'], function ($r) {
    var_dump($r); 
});
```

## **sUnion**

يسترد الأمر اتحاد المجموعات المحددة. تُعامل المجموعات الغير موجودة كمجموعات فارغة.

```php
$redis->sUnion(['s0', 's1', 's2'], function ($r) {
    var_dump($r); // []
});
```

## **sUnionStore**

يقوم بتخزين اتحاد المجموعات المحددة في المجموعة المحددة (destination) ويُرجع عدد العناصر. إذا كانت المجموعة المحددة (destination) موجودة بالفعل ، فسيتم استبدالها.

```php
$redis->del('s0', 's1', 's2');
$redis->sAdd('s0', '1');
$redis->sAdd('s0', '2');
$redis->sAdd('s1', '3');
$redis->sAdd('s1', '1');
$redis->sAdd('s2', '3');
$redis->sAdd('s2', '4');
$redis->sUnionStore('dst', 's0', 's1', 's2', function ($r) {
    var_dump($r); // 4
});
$redis->sMembers('dst', function ($r) {
    var_dump($r); // ['1', '2', '3', '4']
});
```
