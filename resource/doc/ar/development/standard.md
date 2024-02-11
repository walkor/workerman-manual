# مواصفات التطوير

## دليل التطبيق

يمكن وضع دليل التطبيق في أي مكان.

## ملف الدخول

مثل تطبيقات PHP في nginx + PHP-FPM ، تحتاج تطبيقات Workerman أيضًا إلى ملف دخول ، ولا يوجد متطلبات لاسم ملف الدخول ، ويتم تشغيل ملف الدخول بهذه الطريقة بتشغيل PHP Cli.

يحتوي ملف الدخول على الشفرة ذات الصلة بإنشاء العمليات الاستماع ، على سبيل المثال الكود الرئيسي التالي المعتمد على Worker

test.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// إنشاء عامل عمل للاستماع على منفذ 2345 واستخدام بروتوكول http
$http_worker = new Worker("http://0.0.0.0:2345");

// تشغيل 4 عمليات لتقديم الخدمة للمستخدم
$http_worker->count = 4;

// عند استقبال بيانات من المستعرض ، قم بالرد بكلمة "hello world"
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    // إرسال كلمة "hello world" إلى المستعرض
    $connection->send('hello world');
};

Worker::runAll();

```

## مواصفات الكود في Workerman

1. يجب أن تبدأ أسماء الفصول بأحرف كبيرة وفقًا للنمط كامل الجمل. يجب أن يكون اسم ملف الفصل مطابقًا لاسم الفصل الداخلي في الملف ، لضمان التحميل التلقائي. على سبيل المثال:
```php
class UserInfo
{
...
```

2. استخدام مساحة الأسماء ، حيث يجب أن تتطابق أسماء مساحة الأسماء مع مسار الدليل ، ويجب أن تكون السلسلة الجذرية لمستودع المطور القاعدة. على سبيل المثال ، للمشروع MyApp/ ، ملف الفصل MyApp/MyClass.php يجب أن يكون له مساحة أسماء مفقودة. أما ملف الفصل MyApp/Protocols/MyProtocol.php فلأن MyProtocol.php في دليل Protocols لمشروع MyApp ، فيجب إضافة مساحة الأسماء `namespace Protocols;` ، كالتالي:
```php
namespace Protocols;
class MyProtocol
{
.....
```

3. الدوال العادية والمتغيرات تستخدم الأسماء الصغيرة مع إضافة شرطة سفلية ، على سبيل المثال:
```php
$connection_list = array();
function get_connection_list()
{
....
```

4. أعضاء الفصل وأساليب الفصل تستخدم الحالة الصغيرة بأول حرف بأسلوب الجملة الحمراء ، على سبيل المثال:
```php
public $connectionList;
public function getConnectionList();
```

5. المعلمات المرتبطة بدوال أو فصول الفصل تستخدم الأسماء الصغيرة مع إضافة شرطة سفلية
```php
function get_connection_list($one_param, $tow_param)
{
....

```
