## كيفية تخصيص البروتوكول

في الواقع، تخصيص البروتوكول الخاص بك أمر بسيط نسبيًا. البروتوكول البسيط عادةً ما يحتوي على جزأين:
 * تحديد علامات تمييز حدود البيانات
 * تعريف تنسيق البيانات

## مثال

### تعريف البروتوكول
نفترض هنا أن علامة تمييز حدود البيانات هي السطر الجديد "\ n" (يرجى ملاحظة أن البيانات الصادرة نفسها لا يجب أن تحتوي على سطر جديد) ، وتنسيق البيانات هو Json. على سبيل المثال ، هناك بيانات طلب تتوافق مع هذا القاعدة.

<pre>
{"type":"message","content":"hello"}
</pre>

يرجى ملاحظة أن هناك حرف سطر جديد في نهاية بيانات الطلب (يتم تمثيله في PHP كسلسلة "تقويمية مزدوجة" "\n")، ويمثل نهاية الطلب.

### خطوات التنفيذ
إذا كنت ترغب في تنفيذ البروتوكول السابق في Workerman، وفرضا أن اسم البروتوكول هو JsonNL وهو موجود في المشروع الخاص بك MyApp، يجب اتباع الخطوات التالية

1- ضع ملف البروتوكول في مجلد Protocols بالمشروع، على سبيل المثال: ملف MyApp/Protocols/JsonNL.php

2- قم بتنفيذ فئة JsonNL، واستخدم `namespace Protocols;` كمساحة اسمية، يجب عليك تنفيذ ثلاثة طرق استاتيك تسمى input و encode و decode

يرجى ملاحظة: سيقوم Workerman تلقائياً بدعوة هذه الطرق الثلاثة الاستاتيكية لتنفيذ تقسيم الحزم وفك تشفيرها وتعبئتها. يمكنكم الرجوع لتفاصيل العملية من خلال مراجعة تفسير العملية أدناه.

### تفاعل Workerman مع فئة البروتوكول
1- على سبيل المثال، إذا قام العميل بإرسال حزمة بيانات إلى الخادم، سيقوم الخادم فور استلام البيانات (وقد يكون جزئيا) بدعوة فورية للطريقة input للبروتوكول، وذلك لفحص طول حزمة البيانات وإرجاع قيمة الطول `$length` إلى إطار Workerman.

2- بعد استلام إطار Workerman قيمة `$length`، سيقوم بفحص ما إذا كانت طول البيانات في مخزن البيانات الحالي تساوي `$length`، وإذا لم يتسع المخزن فسيظل ينتظر حتى يصبح طول البيانات في المخزن ليس أقل من `$length`.

3- بمجرد أن يكون طول البيانات في المخزن كافيًا، سيقوم Workerman بقص البيانات بطول `$length` من المخزن (أي تقسيم الحزمة)، ومن ثم يدعو طريقة الفك تشفير `decode` للبروتوكول، وبعد فك تشفير البيانات ستكون `$data`.

4- بعد فك تشفير البيانات، سوف يقوم Workerman بنقل البيانات `$data` إلى الأعمال الفرعية من خلال استدعاء `onMessage($connection, $data)`، حيث يمكن للأعمال الفرعية في onMessage استخدام المتغير `$data` للوصول إلى البيانات الكاملة والتي تم فك تشفيرها من قبل العميل.

5- عندما يحتاج الأعمال الفرعية في onMessage إلى إرسال بيانات إلى العميل من خلال استدعاء `$connection->send($buffer)`، سيقوم Workerman تلقائياً باستخدام طريقة الحزم `encode` للبروتوكول لتعبئة `$buffer` ثم يُرسلها إلى العميل.

### التنفيذ الفعلي

**تنفيذ MyApp/Protocols/JsonNL.php**

```php
namespace Protocols;
class JsonNL
{
    /**
     * التحقق من إكتمال الحزمة
     * إذا كان بإمكانك الحصول على طول الحزمة في $recv_buffer، فسيتم إرجاع طول الحزمة بأكملها
     * إذا لم يكن ممكناً الحصول على طول الحزمة في $recv_buffer، فسيتم إرجاع 0 للإنتظار للمزيد من البيانات
     * إذا تم إرجاع قيمة خاطئة أو سالبة، فسيتم فصل الاتصال نتيجة لحدوث خطأ
     * @param ConnectionInterface $connection
     * @param string $recv_buffer
     * @return int|false
     */
    public static function input($recv_buffer, $connection)
    {
        // الحصول على موضع السطر الجديد "\n"
        $pos = strpos($recv_buffer, "\n");
        // إذا لم يوجد سطر جديد، فلا يمكن التعرف على طول الحزمة، وبالتالي يجب الانتظار حتى تصل المزيد من البيانات
        if($pos === false)
        {
            return 0;
        }
        // إذا وجدت سطر جديد، يتم إرجاع طول الحزمة الحالية (بما في ذلك السطر الجديد)
        return $pos+1;
    }

    /**
     * التعبئة، عندما يحتاج إطار العميل إلى إرسال البيانات، سيتم دعوة هذه الطريقة تلقائياً
     * @param string $buffer
     * @return string
     */
    public static function encode($buffer)
    {
        // تسلسل ال Json، وإضافة السطر الجديد لتحديد نهاية الطلب
        return json_encode($buffer)."\n";
    }

    /**
     * فك تشفير، عندما يكون عدد البيانات المستلمة متساوياً مع القيمة المُرجَعة من input (قيمة موجبة)، سيتم استدعاء هذه الطريقة تلقائياً
     * وستُمرر البيانات المُشفرة إلى متغير $data لل,t$callback من onMessage
     * @param string $buffer
     * @return string
     */
    public static function decode($buffer)
    {
        // إزالة السطر الجديد، واستعادتها إلى مصفوفة
        return json_decode(trim($buffer), true);
    }
}
```

وبهذا نكون قد انتهينا من تنفيذ بروتوكول JsonNL ليمكن استخدامه في مشروع MyApp.

يمكنك الاطلاع على الطريقة المستخدمة في المشروع من خلال المثال التالي:

ملف: MyApp\start.php

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$json_worker = new Worker('JsonNL://0.0.0.0:1234');
$json_worker->onMessage = function(TcpConnection $connection, $data) {

    // $data يمثل البيانات التي يرسلها العميل، وقد تم فك تشفيرها بواسطة JsonNL::decode
    echo $data;
    
    // البيانات التي تم إرسالها عبر $connection->send ستُحزم تلقائياً بمساعدة طريقة JsonNL::encode ثم يتم إرسالها إلى العميل
    $connection->send(array('code'=>0, 'msg'=>'ok'));
    
};
Worker::runAll();
```

> **ملحوظة**
> سيحاول Workerman تحميل البروتوكول الموجود تحت مساحة الاسم Protocols، ومثلا `new Worker('JsonNL://0.0.0.0:1234')` سيحاول تحميل بروتوكول `Protocols\JsonNL`.
> إذا كنت تواجه خطأ "Class 'Protocols\JsonNL' not found"، يرجى الاطلاع على [التحميل التلقائي](../faq/autoload.md) لتنفيذ التحميل التلقائي.

### شرح واجهة البروتوكول
يجب على فئة البروتوكول التي يتم تطويرها في Workerman تنفيذ ثلاثة طرق استاتيكية، input و encode و decode. يمكنكم الإطلاع على شرح واجهة عمل لبروتوكول في Workerman/Protocols/ProtocolInterface.php من خلال الرابط التالي:

```php
namespace Workerman\Protocols;

use \Workerman\Connection\ConnectionInterface;

/**
 * Interface ProtocolInterface
 * @author walkor <walkor@workerman.net>
 */
interface ProtocolInterface
{
    /**
     * تقسيم الحزمة في $recv_buffer
     * إذا يمكن الحصول على طول الحزمة في $recv_buffer فسيتم إرجاع طول الحزمة الكاملة
     * وإذا لم يكن ممكناً الحصول على طول الحزمة سيتم إرجاع 0 للإنتظار للمزيد من البيانات
     * وإذا تم إرجاع قيمة خاطئة أو سالبة يعني أن هناك خطأ ويجب فصل الاتصال
     *
     * @param ConnectionInterface $connection
     * @param string $recv_buffer
     * @return int|false
     */
    public static function input($recv_buffer, ConnectionInterface $connection);

    /**
     * فك تشفير الحزمة
     *
     * إذا كان قيمة الطول المُرجعة من طريقة input أكبر من 0، وٌاستلم Workerman كمية كافية من البيانات
     * فسيقوم بدعوة هذه الطريقة تلقائياً لفك تشفير البيانات، وإرسالها من خلال callback onMessage
     * وهذا يعني أنه عند استقبال طلب كامل من العميل سيتم فك تشفيره تلقائياً دون الحاجة لإدخال طلب التوقف بشكل يدوي في الشفرة
     *
     * @param ConnectionInterface $connection
     * @param string $recv_buffer
     */
    public static function decode($recv_buffer, ConnectionInterface $connection);

    /**
     * تعبئة الحزمة
     *
     * عندما يحتاج إطار العميل إلى إرسال البيانات سيقوم Workerman بتعبئة $data تلقائياً من خلال هذه الطريقة
     * وذلك بحيث تتماشى البيانات مع نمط البروتوكول، ثم إرسالها إلى العميل
     *
     * @param ConnectionInterface $connection
     * @param mixed $data
     */
    public static function encode($data, ConnectionInterface $connection);
}
```

## ملاحظة:
لا تتطلب Workerman أن تكون فئة البروتوكول مبنية على ProtocolInterface بشكل صارم. في الواقع، يجب أن تحتوي فقط فئة البروتوكول على ثلاثة طرق استاتيكية input و encode و decode لتكون صالحة.
