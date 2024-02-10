# التحميل التلقائي

إذا لم يكن لمشروعك تحميل تلقائي محدد، يمكنك استخدام مُنتج Composer لتوليده، وفيما يلي الخطوات المطلوبة.

**1. تعديل ملف composer.json**  
أضف autoload.psr-4 إلى ملف composer.json بالشكل التالي `"" : "./"`، كمثال:
```json
{
    "require": {
        "workerman/workerman" : "^v4.0.0"
    },
    "autoload": {
        "psr-4": {
            "" : "./"
        }
    }
}
```

**2. توليد autoload.php**  
قم بتشغيل الأمر `composer dumpautoload`

**3. تحميل autoload.php**  
قم بتحميل ملف `vendor/autoload.php` في رأس ملف start.php للمشروع، على سبيل المثال:
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
```

**4. إعادة تشغيل workerman**  
أعد تشغيل workerman بواسطة الأمر التالي `php start.php restart`
