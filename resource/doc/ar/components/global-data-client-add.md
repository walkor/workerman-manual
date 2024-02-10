# إضافة
**``` (متطلب Workerman >= 3.3.0) ```**

```php
bool \GlobalData\Client::add(string $key, mixed $value)
```
الإضافة الذرية. إذا كان المفتاح موجودًا بالفعل ، ستُرجع القيمة false.

## المعاملات

``` $key ```

المفتاح. (على سبيل المثال```$global->abc```، ```abc``` هو المفتاح)

``` $value ```

القيمة المخزنة.

## العائد
ترجع true في حال نجاح العملية، وإلا ترجع false.

## مثال

```php
$global = new GlobalData\Client('127.0.0.1:2207');

if($global->add('some_key', 10))
{
    // تم تعيين $global->some_key بنجاح
    echo "add success " , $global->some_key;
}
else
{
    // تم إنشاء $global->some_key بالفعل، فشلت العملية
    echo "add fail " , var_export($global->some_key);
}
```
