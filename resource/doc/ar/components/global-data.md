# مكون GlobalData لمشاركة المتغيرات بشكل عام
**``` (متطلب Workerman الإصدار>=3.3.0) ```**

رابط المصدر: https://github.com/walkor/GlobalData

## ملاحظة
يتطلب GlobalData إصدار Workerman>=3.3.0

## التثبيت

`composer require workerman/globaldata`

## ال原理

يُستخدم معه أساليب سحرية في PHP مثل ```__set __get __isset __unset``` لتنشيط اتصال مع خادم GlobalData، حيث يتم تخزين المتغيرات بشكل فعلي في خادم GlobalData. على سبيل المثال، عندما يتم تعيين خاصية غير موجودة لفئة العميل، سينشط طريقة ```__set``` السحرية، حيث ستقوم فئة العميل بإرسال طلب إلى خادم GlobalData لتخزين المتغير. وعند الوصول إلى متغير غير موجود في فئة العميل، ستنشط طريقة ```__get``` للفئة حيث سيقوم العميل بإرسال طلب إلى خادم GlobalData لقراءة هذا القيمة، وبالتالي يتم إكمال مشاركة المتغيرات بين العمليات.

```php
require_once __DIR__ . '/vendor/autoload.php';

// الاتصال بخادم Global Data
$global = new GlobalData\Client('127.0.0.1:2207');

// تنشيط $global->__isset('somedata') للاستعلام ما إذا كان الخادم يخزن مفتاحًا يحمل القيمة somedata
isset($global->somedata);

// تنشيط $global->__set('somedata',array(1,2,3))، لتخزين القيمة المقابلة لـ somedata كـ array(1,2,3)
$global->somedata = array(1,2,3);

// تنشيط $global->__get('somedata')، للاستعلام عن القيمة المقابلة لـ somedata من الخادم
var_export($global->somedata);

// تنشيط $global->__unset('somedata')، لإبلاغ الخادم بحذف somedata والقيمة المقابلة لها
unset($global->somedata);
```
