وأمثلة

## الامثلة الأول

### تعريف البروتوكول
  * الرأس ثابت بطول 10 بايت لحفظ طول حزمة البيانات بأكملها ، وفي حالة عدم كفاية الأرقام ، قم بملء الصفر
  * تنسيق البيانات هو xml

### عينة الحزمة
```xml
0000000121<?xml version="1.0" encoding="ISO-8859-1"?>
<request>
    <module>user</module>
    <action>getInfo</action>
</request>
```
حيث 0000000121 تمثل طول حزمة البيانات بأكملها ، تليها محتوى البيانات بتنسيق xml

### تنفيذ البروتوكول
```php
namespace Protocols;
class XmlProtocol
{
    public static function input($recv_buffer)
    {
        if(strlen($recv_buffer) < 10)
        {
            // ليست كافية 10 بايت ، العودة 0 للانتظار على المزيد من البيانات
            return 0;
        }
        // الطول الإجمالي يحتوي على طول الرأس + طول الحزمة
        $total_len = base_convert(substr($recv_buffer, 0, 10), 10, 10);
        return $total_len;
    }

    public static function decode($recv_buffer)
    {
        // جسم الطلب
        $body = substr($recv_buffer, 10);
        return simplexml_load_string($body);
    }

    public static function encode($xml_string)
    {
        // طول الحزمة + طول الرأس
        $total_length = strlen($xml_string)+10;
        // جعل الأرقام 10 بايت ، وإذا لم يكن كافياً ، قم بملء الصفر
        $total_length_str = str_pad($total_length, 10, '0', STR_PAD_LEFT);
        // ارجاع البيانات
        return $total_length_str . $xml_string;
    }
}
```

## الامثلة الثانية

### تعريف البروتوكول
  * الرأس 4 بايت ترتيب البايت على الشبكة كـ unsigned int ، تعلن عن طول الحزمة بأكملها
  * البيانات عبارة عن سلسلة Json

### عينة الحزمة
<pre>
****{"type":"message","content":"hello all"}
</pre>

حيث الأربعة بايت الأولى تمثل ترتيب البايت على الشبكة unsigned int وهي حروف غير قابلة للرؤية ، تليها بشكل فوري سلسلة البيانات بتنسيق Json

### تنفيذ البروتوكول
```php
namespace Protocols;
class JsonInt
{
    public static function input($recv_buffer)
    {
        // البيانات المستقبلة لا تزال غير كافية بطول 4 بايت ، من المستحيل معرفة طول الحزمة ، العودة 0 للانتظار على المزيد من البيانات
        if(strlen($recv_buffer)<4)
        {
            return 0;
        }
        // تحويل الأربعة بايت الأولى إلى رقم باستخدام دالة unpack ، حيث أن الأربعة بايت الأولى هي طول الحزمة بأكملها
        $unpack_data = unpack('Ntotal_length', $recv_buffer);
        return $unpack_data['total_length'];
    }

    public static function decode($recv_buffer)
    {
        // إزالة الأربعة بايت الأولى ، والحصول على سلسلة البيانات
        $body_json_str = substr($recv_buffer, 4);
        // فك تشفير الجيسون
        return json_decode($body_json_str, true);
    }

    public static function encode($data)
    {
        // تشفير Json للحصول على جسم الحزمة
        $body_json_str = json_encode($data);
        // حساب طول الحزمة بأكملها ، الأربعة بايت الأولى + عدد البايت في السلسلة
        $total_length = 4 + strlen($body_json_str);
        // ارجاع البيانات المحزمة
        return pack('N',$total_length) . $body_json_str;
    }
}
```

## الامثلة الثالثة (استخدام البروتوكول الثنائي لرفع الملفات)

### تعريف البروتوكول

```C
struct
{
  unsigned int total_len;  // طول الحزمة بأكملها ، بترتيب البايت الكبير على الشبكة
  char         name_len;   // طول اسم الملف
  char         name[name_len]; // اسم الملف
  char         file[total_len - BinaryTransfer::PACKAGE_HEAD_LEN - name_len]; // بيانات الملف
}
```
### عينة البروتوكول

<pre> *****logo.png****************** </pre>

حيث الأربعة بايت الأولى * تمثل ترتيب البايت الكبير على الشبكة unsigned int ، وهي حروف غير قابلة للرؤية ، الرقم الخامس * يقوم بتخزين طول اسم الملف في بيت واحد ، تليه اسم الملف ، وتليها بيانات الملف الثنائية الأصلية

### تنفيذ البروتوكول
```php
namespace Protocols;
class BinaryTransfer
{
    // طول الرأس للبروتوكول
    const PACKAGE_HEAD_LEN = 5;

    public static function input($recv_buffer)
    {
        // اذا لم تكن البيانات كافية لطول الرأس ، الاستمرار في الانتظار
        if(strlen($recv_buffer) < self::PACKAGE_HEAD_LEN)
        {
            return 0;
        }
        // فك الحزمة
        $package_data = unpack('Ntotal_len/Cname_len', $recv_buffer);
        // ارجاع طول الحزمة
        return $package_data['total_len'];
    }


    public static function decode($recv_buffer)
    {
        // فك الحزمة
        $package_data = unpack('Ntotal_len/Cname_len', $recv_buffer);
        // طول اسم الملف
        $name_len = $package_data['name_len'];
        // قص اسم الملف من التدفق
        $file_name = substr($recv_buffer, self::PACKAGE_HEAD_LEN, $name_len);
        // قص بيانات الملف الثنائية من التدفق
        $file_data = substr($recv_buffer, self::PACKAGE_HEAD_LEN + $name_len);
         return array(
             'file_name' => $file_name,
             'file_data' => $file_data,
         );
    }

    public static function encode($data)
    {
        // يمكن ترميز البيانات المرسلة إلى العميل حسب الحاجة ، هنا نرجع النص كما هو
        return $data;
    }
}
```

### مثال على استخدام البروتوكول في الخادم

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('BinaryTransfer://0.0.0.0:8333');
// حفظ الملف في tmp
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $save_path = '/tmp/'.$data['file_name'];
    file_put_contents($save_path, $data['file_data']);
    $connection->send("upload success. save path $save_path");
};

Worker::runAll();
```

### مثال على استخدام العميل للملف client.php (هنا يتم استخدام الـ php كعميل لرفع الملف)
```php
<?php
/** عميل رفع الملفات **/
// عنوان الرفع
$address = "127.0.0.1:8333";
// تحقق من معلمات مسار الرفع
if(!isset($argv[1]))
{
   exit("use php client.php \$file_path\n");
}
// مسار الرفع
$file_to_transfer = trim($argv[1]);
// الملف المطلوب رفعه غير موجود محلياً
if(!is_file($file_to_transfer))
{
    exit("$file_to_transfer not exist\n");
}
// بناء اتصال socket
$client = stream_socket_client($address, $errno, $errmsg);
if(!$client)
{
    exit("$errmsg\n");
}
// جعلها تعمل بشكل متزامن
stream_set_blocking($client, 1);
// اسم الملف
$file_name = basename($file_to_transfer);
// طول اسم الملف
$name_len = strlen($file_name);
// بيانات الملف الثنائية
$file_data = file_get_contents($file_to_transfer);
// طول الرأس للبروتوكول 4 بايت لطول الحزمة + بيت واحد لطول اسم الملف
$PACKAGE_HEAD_LEN = 5;
// الحزمة البروتوكولية
$package = pack('NC', $PACKAGE_HEAD_LEN  + strlen($file_name) + strlen($file_data), $name_len) . $file_name . $file_data;
// رفع الملف
fwrite($client, $package);
// طباعة النتيجة
echo fread($client, 8192),"\n";
```


### مثال على استخدام العميل
تشغيل في سطر الأوامر ```php client.php <مسار الملف>```

مثلاً ```php client.php abc.jpg```
## المثال الرابع (تحميل الملفات باستخدام بروتوكول النصوص)

### تعريف البروتوكول

json + سطر جديد، يحتوي الjson على اسم الملف وتشفير البيانات الخاصة بالملف بواسطة base64_encode (سيزيد من الحجم بمقدار 1/3)

### عينة للبروتوكول

{"file_name":"logo.png","file_data":"PD9waHAKLyo......"}\n

يرجى ملاحظة أنه في النهاية، هناك سطر جديد. في PHP يتم تمثيله برمز الاقتباس المزدوج ```"\n"```

### تنفيذ البروتوكول

```php
namespace Protocols;
class TextTransfer
{
    public static function input($recv_buffer)
    {
        $recv_len = strlen($recv_buffer);
        if($recv_buffer[$recv_len-1] !== "\n")
        {
            return 0;
        }
        return strlen($recv_buffer);
    }

    public static function decode($recv_buffer)
    {
        // فك التعبئة
        $package_data = json_decode(trim($recv_buffer), true);
        // استخراج اسم الملف
        $file_name = $package_data['file_name'];
        // استخراج البيانات المشفرة بعد base64_encode
        $file_data = $package_data['file_data'];
        // base64_decode لاستعادة البيانات الثنائية الأصلية للملف
        $file_data = base64_decode($file_data);
        // إرجاع البيانات
        return array(
             'file_name' => $file_name,
             'file_data' => $file_data,
         );
    }

    public static function encode($data)
    {
        // يمكن ترميز البيانات التي سيتم إرسالها إلى العميل حسب الحاجة، ولكن في هذا المكان سنقوم بإعادتها كنص
        return $data;
    }
}
```

### مثال على استخدام بروتوكول الخادم

ملاحظة: الكتابة مشابهة لكتابة تحميل الملفات الثنائية، مما يعني أنه يمكن تبديل البروتوكول تقريباً دون الحاجة إلى تغيير أي كود تجاري

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('TextTransfer://0.0.0.0:8333');
// حفظ الملف في tmp
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $save_path = '/tmp/'.$data['file_name'];
    file_put_contents($save_path, $data['file_data']);
    $connection->send("تم التحميل بنجاح. مسار الحفظ: $save_path");
};

Worker::runAll();
```

### مثال على استخدام العميل textclient.php (هنا يتم استخدام PHP كمحاكي للعميل للتحميل)

```php
<?php
/** عميل تحميل الملفات **/
// عنوان التحميل
$address = "127.0.0.1:8333";
// التحقق من وجود مسار الملف المراد تحميله كوسيطة للأمر
if(!isset($argv[1]))
{
   exit("استخدم php client.php \$file_path\n");
}
// مسار الملف المراد تحميله
$file_to_transfer = trim($argv[1]);
// إذا كان الملف المراد تحميله غير موجود
if(!is_file($file_to_transfer))
{
    exit("$file_to_transfer غير موجود\n");
}
// إقامة اتصال مأخوذ
$client = stream_socket_client($address, $errno, $errmsg);
if(!$client)
{
    exit("$errmsg\n");
}
stream_set_blocking($client, 1);
// اسم الملف
$file_name = basename($file_to_transfer);
// بيانات الملف الثنائية
$file_data = file_get_contents($file_to_transfer);
// تشفير base64
$file_data = base64_encode($file_data);
// الحزمة البيانية
$package_data = array(
    'file_name' => $file_name,
    'file_data' => $file_data,
);
// الحزمة بروتوكول json + سطر جديد
$package = json_encode($package_data)."\n";
// تنفيذ التحميل
fwrite($client, $package);
// طباعة النتيجة
echo fread($client, 8192),"\n";
```

### مثال على استخدام العميل

تشغيل الأمر في سطر الأوامر ```php textclient.php <مسار الملف>```

مثال: ```php textclient.php abc.jpg```
