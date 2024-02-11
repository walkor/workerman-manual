# متطلبات البيئة

## لمستخدمي Windows
ابتداءً من الإصدار 3.5.3، أصبح من الممكن لـ workerman دعم أنظمة Linux و Windows معاً.

1. PHP>=5.4 مطلوبة، ويجب تكوين متغيرات البيئة الخاصة بـ PHP.

2. إصدار Windows من Workerman لا يعتمد على أية مكونات إضافية.

3. لمعرفة كيفية التثبيت والقيود المفروضة على الاستخدام [**هنا**](https://www.workerman.net/windows).

4. نظرًا لوجود العديد من القيود على استخدام Workerman في نظام Windows، يُفضل في البيئة الإنتاجية استخدام نظام Linux، ويُنصح فقط باستخدام نظام Windows في بيئة التطوير.

``` ====  هذه الصفحة مخصصة فقط لمستخدمي Linux، يُرجى تجاهلها إن كنت تستخدم Windows. =====```

## لمستخدمي Linux (بما في ذلك نظام Mac OS)
يمكن لمستخدمي نظام Linux استخدام الإصدار الخاص بنظام Linux من Workerman فقط.

1. يجب تثبيت PHP>=5.4 والتأكد من تثبيت مكونات pcntl و posix.

2. من الأفضل تثبيت مكون event، ولكن ليس بشكل إلزامي (يرجى ملاحظة أن مكون event يتطلب PHP>=5.4).

### سكريبت فحص البيئة لنظام Linux
يُمكن لمستخدمي نظام Linux تشغيل السكريبت التالي لفحص ما إذا كانت البيئة المحلية تستوفي متطلبات Workerman

```curl -Ss https://www.workerman.net/check | php```

إذا كانت جميع الإشعارات في السكريبت تظهر "ok"، فإن ذلك يعني توافر بيئة تشغيل Workerman

(يرجى ملاحظة: السكريبت لا يقوم بفحص مكون event، وفي حال كان عدد الاتصالات المتزامنة أكبر من 1024، فإنه يُوصى بتثبيت مكون event. يُرجى الرجوع إلى الجزء التالي لمعرفة كيفية التثبيت)

## تفاصيل الشرح

### حول PHP-CLI

يتم تشغيل Workerman بناءً على [PHP Command Line (PHP-CLI)](https://php.net/manual/zh/features.commandline.php). يُعتبر PHP-CLI برنامجًا قابلاً للتنفيذ مستقلًا يعمل بشكل منفصل وغير تعتمد على PHP-FPM أو PHP MOD-PHP في Apache، ولا يتداخل معها ولا يعتمد عليها.

### حول المكونات الضرورية لـ Workerman

1. [مكون pcntl](https://cn2.php.net/manual/zh/book.pcntl.php)

مكون pcntl هو مكون هام لتحكم العمليات في PHP في بيئة Linux. يستخدم Workerman هذا المكون لميزات مثل [إنشاء العمليات](https://cn2.php.net/manual/zh/function.pcntl-fork.php) و[التحكم بالإشارات](https://cn2.php.net/manual/zh/function.pcntl-signal.php) و[المؤقتات](https://cn2.php.net/manual/zh/function.pcntl-alarm.php) و[رصد حالة العملية](https://cn2.php.net/manual/zh/function.pcntl-waitpid.php). لا يدعم هذا المكون نظام Windows.

2. [مكون posix](https://cn2.php.net/manual/zh/book.posix.php)

يتيح مكون posix لـ PHP في نظام Linux استدعاء واجهات النظام المقدمة من خلال [معيار POSIX](https://baike.baidu.com/view/209573.htm). يستخدم Workerman هذا المكون بشكل رئيسي لتحقيق ميزات مثل تحويل العملية إلى وضع الخلفية والتحكم في مجموعة المستخدمين وما إلى ذلك. لا يدعم هذا المكون نظام Windows.

3. [مكون Event](https://php.net/manual/zh/book.event.php) أو [مكون libevent](https://cn2.php.net/manual/en/book.libevent.php)

يمكن لمكون event أن يتيح لـ PHP استخدام آلية معالجة الأحداث المتقدمة مثل Epoll و Kqueue التي توفرها النظام. يمكن أن تحسن هذه الآليات أداء Workerman في استخدام الاتصالات المتزامنة والمرتبة. إذا لم يتم تثبيته، سيتم استخدام آلية معالجة الأحداث الأصلية في PHP بشكل افتراضي.

## كيفية تثبيت المكونات

يرجى الرجوع إلى الفصل [تثبيت المكونات](../appendices/install-extension.md)
