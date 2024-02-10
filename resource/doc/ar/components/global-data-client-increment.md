# الزيادة
**``` (يتطلب Workerman الإصدار >= 3.3.0) ```**
```php
bool \GlobalData\Client::increment(string $key[, int $step = 1])
```
الزيادة الذرية. تزيد قيمة العنصر الرقمي بالحجم المحدد في المعامل step. إذا لم تكن قيمة العنصر نوع رقمي، ستعامل كقيمة 0 قبل الزيادة. إذا لم يكن العنصر موجودًا، ستُرجع false.

## المعاملات

 ``` $key ```

المفتاح. (على سبيل المثال ```$global->abc```، فإن ```abc``` هو المفتاح)


 ``` $value ```

الحجم الذي يجب زيادة قيمة العنصر به.

## القيمة المُرجَعة
ترجع القيمة true في حال النجاح، وإلا ترجع القيمة false.

## مثال

```php
$global = new GlobalData\Client('127.0.0.1:2207');

$global->some_key = 0;

// زيادة غير ذرية
$global->some_key++;

echo $global->some_key."\n";

// الزيادة الذرية
$global->increment('some_key');

echo $global->some_key."\n";
```
