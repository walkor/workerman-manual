# دليل التثبيت
Workerman في الواقع هو حزمة كود PHP، إذا كنت قد قمت بتثبيت بيئة PHP بالفعل، فما عليك سوى تحميل كود المصدر الخاص بـ Workerman أو العرض التوضيحي وبعد ذلك يمكنك تشغيلها.

**تثبيت Composer:**
```sh
composer require workerman/workerman
```

> **ملاحظة**
> بعض المرايا البديلة لـ composer غير كاملة، استخدم الأمر التالي `composer config -g --unset repos.packagist` لإزالة المرايا البديلة.

# مستخدمي الويندوز (مهم)

ابتداءً من الإصدار 3.5.3 من Workerman، أصبحت Workerman تدعم بشكل فعال نظامي التشغيل ويندوز ولينكس.
مستخدمي ويندوز بحاجة إلى تهيئة متغيرات بيئة PHP.

 ` === ينطبق ما بعد هذا السطر فقط على بيئة لينكس Workerman، يرجى تجاهلها إذا كنت مستخدمًا لويندوز === `

# فحص بيئة نظام التشغيل لينكس
يمكن استخدام النص التالي لاختبار ما إذا كانت بيئة PHP المحلية تفي بمتطلبات تشغيل Workerman.
 `curl -Ss https://www.workerman.net/check | php`

إذا ظهرت جميع النتائج "موافق"، فهذا يعني أن تلبية متطلبات Workerman، يمكنك تحميل مثال من [الموقع الرسمي](https://www.workerman.net/) مباشرة للتشغيل.

إذا لم تظهر جميع النتائج "موافق"، فيمكنك الرجوع إلى الوثائق أدناه لتثبيت الامتدادات المفقودة.

(ملاحظة: البرنامج النصي للفحص لم يفحص امتداد event، إذا كان عدد الاتصالات المتزامنة للعمل كبيرًا، يجب تثبيت امتداد event، و[تحسين نواة Linux](../appendices/kernel-optimization.md)، يمكنك الرجوع إلى الدليل أدناه لتعلم كيفية تثبيت الامتداد)

# تثبيت الأمتدادات المفقودة لبيئة نظام التشغيل لينكس

## تثبيت الأمتدادات pcntl و posix:
**لنظام CentOS**
إذا كان PHP تم تثبيته عبر yum، يمكنك تثبيت الأمتدادات pcntl و posix بالأمر التالي في سطر الأوامر: ```yum install php-process```.


إذا فشل التثبيت أو إذا لم يتم تثبيت PHP باستخدام yum، يرجى الرجوع إلى الدليل [الملحق-تثبيت الأمتدادات](../appendices/install-extension.md) في الطريقة الثالثة التثبيت بتركيب الشفرة المصدرية.

**لنظام Debian/Ubuntu/Mac OS**
يرجى الرجوع إلى الدليل [الملحق-تثبيت الأمتدادات]((../appendices/install-extension.md)) في الطريقة الثالثة التثبيت بتركيب الشفرة المصدرية.

## تثبيت الأمتداد event:
لدعم عدد أكبر من الاتصالات المتزامنة، يجب تثبيت امتداد event و[تحسين نواة Linux](../appendices/kernel-optimization.md)، ويمكن التثبيت كما يلي:

**لنظام CentOS**

1. قم بتثبيت حزمة libevent-devel التي يعتمد عليها امتداد event، في سطر الأوامر:
```shell
yum install libevent-devel -y
# إذا فشل التثبيت، جرّب الأمر التالي
# yum install libevent2-devel -y
```

2. قم بتثبيت امتداد event، في سطر الأوامر:
(يتطلب امتداد event PHP>=5.4)
```shell
pecl install event
```
يرجى تجاهل الإشعار:```Include libevent OpenSSL support [yes] :``` واضغط على enter، ثم اضغط enter الباقي.

3. قم بتشغيل ```php --ini``` للعثور على ملف php.ini وافتحه وأضف الضبط التالي في السطر الأخير:
```shell
extension=event.so
```

**لنظام Debian/Ubuntu**

1. يرجى تثبيت الحزمة libevent-dev التي يعتمد عليها امتداد event، في سطر الأوامر:
```shell
apt-get install libevent-dev -y
# إذا فشل التثبيت، جرّب الأمر التالي
# apt-get install libevent2-dev -y
```

2. قم بتثبيت امتداد event، في سطر الأوامر:
```shell
pecl install event
```
يرجى تجاهل الإشعار:```Include libevent OpenSSL support [yes] :``` واضغط على enter، ثم اضغط enter الباقي.

3. قم بتشغيل ```php --ini``` للعثور على ملف php.ini وافتحه وأضف الضبط التالي في السطر الأخير:
```shell
extension=event.so
```

**دليل تثبيت نظام Mac OS**

نظام Mac عادةً ما يستخدم كجهاز تطويري، ولا يجب تثبيت امتداد event.

# تثبيت النظام الكامل الجديد (تثبيت PHP + الأمتدادات)

## دليل تثبيت نظام CentOS

1. في سطر الأوامر، قم بتشغيل الأمر التالي (يتضمن تثبيت برنامج php-cli الرئيسي وأيضًا امتدادات pcntl، posix، libevent وبرنامج git):
```shell
yum install php-cli php-process git gcc php-devel php-pear libevent-devel -y
```

2. قم بتثبيت امتداد event، في سطر الأوامر:
(موضوع: امتداد event يتطلب PHP>=5.4)
```shell
pecl install event
```
يرجى تجاهل الإشعار:```Include libevent OpenSSL support [yes] :``` واضغط على enter، ثم اضغط enter الباقي.

3. قم بتشغيل ```php --ini``` للعثور على ملف php.ini وافتحه وأضف الضبط التالي في السطر الأخير:
```shell
extension=event.so
```

4. في سطر الأوامر، قم بتشغيل الأمر التالي (يتضمن تنزيل البرنامج الرئيسي لـ Workerman من GitHub):
```shell
git clone https://github.com/walkor/Workerman
```

5. قم بالرجوع إلى [دليل البدء - جزء مثال بسيط للتطوير](../getting-started/simple-example.md) للبدء بكتابة ملف الإدخال وتشغيله.
أو قم بتنزيل العرض التوضيحي الجاهز من [الموقع الرسمي](https://www.workerman.net/) للتشغيل.
## دليل تثبيت نظام Debian/Ubuntu

1. تشغيل الأمر في سطر الأوامر (هذا الخطوة تحتوي على تثبيت برنامج php-cli الرئيسي ومكتبة libevent وبرنامج git):
```shell
apt-get install php-cli git gcc php-pear php-dev libevent-dev -y
```

2. تثبيت امتداد event، تشغيل الأمر في سطر الأوامر
(تنبيه: امتداد event يتطلب PHP >=5.4)
```shell
pecl install event
```
تنبيه: عندما تظهر الرسالة `Include libevent OpenSSL support [yes] :`، أدخل `no` واضغط على Enter، وإلا فقط اضغط على Enter.

3. تشغيل `php --ini` للعثور على ملف php.ini وفتحه، ثم أضف الضبط التالي في السطر الأخير:
```shell
extension=event.so
```

4. تشغيل الأمر في سطر الأوامر (هذه الخطوة تقوم بتنزلية برنامج Workerman الرئيسي عبر جيثب)
```shell
git clone https://github.com/walkor/Workerman
```

5. يمكنك الرجوع إلى [دليل البدء - جزء المثال البسيط](../getting-started/simple-example.md) لكتابة ملف الدخول وتشغيله.
أو يمكنك تنزيل المثال الجاهز من [الموقع الرسمي](https://www.workerman.net/).

## دليل تثبيت نظام macOS
**الطريقة الأولى:** نظام macOS يحتوي على PHP Cli من تلقاء نفسه، لكنه قد يكون ناقصًا من امتداد pcntl.

1. يمكنك الرجوع إلى دليل [المرفق - تثبيت الامتداد](../appendices/install-extension.md) في الجزء الثالث لتثبيت pcntl امتداد.

2. يمكنك الرجوع إلى دليل [المرفق - تثبيت الامتداد](../appendices/install-extension.md) في الجزء الرابع لتثبيت امتداد event باستخدام phpize (يمكن تجاهل هذه الخطوة إذا كنت تستخدم جهاز تطوير فقط).

3. يمكنك تنزيل برنامج Workerman الرئيسي عبر https://www.workerman.net/download/workermanzip، أو يمكنك تنزيل مثال من [الموقع الرسمي](https://www.workerman.net/).

**الطريقة الثانية:** تثبيت php والامتدادات المتوافقة عبر أمر "brew".

1. تشغيل الأوامر التالية في سطر الأوامر لتثبيت أداة "brew" (إذا كنت قد قمت بتثبيت "brew" من قبل، يمكنك تخطي هذه الخطوة)
```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

2. تشغيل الأوامر التالية في سطر الأوامر لتثبيت "php"
```shell
brew install php
```

3. تشغيل الأوامر التالية في سطر الأوامر لتثبيت امتداد "event"
```shell
brew install php-event    
```

4. يمكنك تنزيل مثال من [الموقع الرسمي](https://www.workerman.net/).

# شرح امتداد Event
يمكن عدم تثبيت [امتداد Event](https://php.net/manual/zh/book.event.php) إذا لم يكن هناك حاجة لدعم أكثر من 1000 اتصال متزامن. ولكنه يوصى به عند الحاجة إلى دعم أعداد كبيرة من الاتصالات المتزامنة. إذا كانت أعداد الاتصالات المتزامنة منخفضة، مثل أقل من 1000 اتصال متزامن، فيمكن عدم تثبيته.

## الأسئلة الشائعة
1. إذا واجهت خطأ يشبه الآتي `checking for include/event2/event.h... not found`، فيرجى محاولة حذف مكتبة libevent-dev وتثبيت libevent2-dev بدلاً منها.
نظام CentOS: yum remove libevent-devel && yum install libevent2-devel
نظام Debian/Ubuntu: apt-get remove libevent-dev && apt-get install libevent2-dev

2. إذا واجهت خطأ يشبه الآتي `NOTICE: PHP message: PHP Warning: PHP Startup: Unable to load dynamic library '.../event.so' - ..../event.so: undefined symbol: php_sockets_le_socket in Unknown on line 0`.
يرجى تغيير ترتيب تحميل event.so و socket.so، أي في php.ini، قم بكتابة `extension=socket.so` قبل `extension=event.so`، لتحميل امتداد socket أولاً.
