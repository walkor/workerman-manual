# فشل تشغيل workerman

## ظاهرة 1
بعد البدء، يظهر خطأ مماثل لما يلي:
```php
php start.php start
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxxx (Address already in use) in ...workerman/Worker.php on line xxxx
```
**الكلمات الرئيسية:** ```Address already in use```

**السبب الجذري:** استخدام المنفذ، لا يمكن البدء.

#### الحل 1

يمكنك استخدام الأمر ```netstat -anp | grep رقم المنفذ``` لمعرفة أي برنامج يستخدم المنفذ.
ثم قم بإيقاف البرنامج المقابل لتحرير المنفذ.

#### الحل 2
إذا لم يتمكن أي برنامج من إيقاف استخدام المنفذ، يمكنك حل المشكلة عن طريق تغيير منفذ workerman.

#### الحل 3
إذا كان workerman يستخدم المنفذ، ولا يمكن إيقافه باستخدام الأمر stop (وعادةً ما يكون بسبب فقدان ملف pid أو إيقاف العملية الرئيسية من قبل المطور)، يمكنك قتل عملية workerman بتشغيل الأمرين التاليين.
``` 
killall php
ps aux|grep -i workerman|awk '{print $2}'|xargs kill -9
```

#### الحل 4
إذا لم يكن هناك برنامج يستمع على هذا المنفذ، فقد يكون المطور قام بإعداد استماع لمنفذين أو أكثر وهما متطابقين، يجب على المطور التحقق بنفسه من ملف البدء إذا كان يستمع إلى نفس المنفذ.

#### الحل 5
تحقق مما إذا كان البرنامج قد فتح قدرة إعادة استخدام المنفذ (reusePort)، جرب إغلاق هذه الخاصية.

## ظاهرة 2
بعد البدء، يظهر خطأ مماثل لما يلي:
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxx (Cannot assign requested address) in ...workerman/Worker.php on line xxxx
```
أو
``` 
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxxx (في سياقه، العنوان المطلوب غير صالح) in ...workerman/Worker.php on line xxxx
```
**الكلمات الرئيسية:** `Cannot assign requested address` أو `العنوان المطلوب غير صالح`

**سبب الفشل:** 	
خطأ في إدخال عنوان ip المستمع، ليس عنوان ip المضيف الخاص.

**نصيحة:** يمكن لنظام Linux استخدام الأمر ```ifconfig``` لعرض جميع عناوين IP للبطاقة  الشبكية. إذا كنت مستخدمًا لخادم سحابي (مثل خدمة الأليكسا أو تينسنت كلاود)، فضلًا أن تلاحظ أن عنوان IP العام الخاص بك قد يكون عنوان IP استئجاري (كما في شبكة أليكسا للخوادم الخاصة)، وبالتالي لا ينتمي عنوان IP العام إلى الخادم الحالي، لذلك لا يمكن الاستماع من خلاله. ومع ذلك، فلا تزال يمكنك استخدام 0.0.0.0 للربط.

## ظاهرة 3
```php
Waring stream_socket_server has been disabled for security reasons in ...
```
**سبب الفشل:** تم تعطيل دالة stream_socket_server في ملف php.ini.

**طريقة الحل**

1. قم بتشغيل الأمر ```php --ini``` للعثور على ملف php.ini.
2. افتح ملف php.ini وابحث عن عنصر disable_functions، ثم قم بحذف عنصر stream_socket_server المحظور.

## ظاهرة 4
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://0.0.0.0:xxx (Permission denied)
```
**سبب الفشل:**	
في نظام لينكس، إذا كانت الاستماع على المنفذ أقل من 1024، يجب أن يكون لديك صلاحيات المستخدم الجذر.

**طريقة الحل**

استخدام منفذ أكبر من 1024 أو استخدام حساب المستخدم الجذر لتشغيل الخدمة.
