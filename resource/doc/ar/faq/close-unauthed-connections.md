اغلاق الاتصالات غير المصادق عليها

**السؤال:**
كيف يمكنني إغلاق العميل الذي لم يرسل أي بيانات في الوقت المحدد، مثلاً إذا لم يتلقى أي بيانات في غضون 30 ثانية فيجب إغلاق هذا العميل تلقائيًا، الهدف هو جعل الاتصالات غير المصادق عليها يجب أن تتم المصادقة في الوقت المحدد

**الجواب:**

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('xxx://x.x.x.x:x');
$worker->onConnect = function(TcpConnection $connection)
{
    // يتم إضافة خاصية auth_timer_id المؤقتة لكائن $connection لتخزين معرف المؤقت
    // إغلاق الاتصال بعد 30 ثانية، يجب على العميل إزالة المؤقت في غضون 30 ثانية
    $connection->auth_timer_id = Timer::add(30, function()use($connection){
        $connection->close();
    }, null, false);
};
$worker->onMessage = function(TcpConnection $connection, $msg)
{
    $msg = json_decode($msg, true);
    switch($msg['type'])
    {
    case 'login':
        ...تم تجاهله
        // التحقق ناجح، إزالة المؤقت لمنع إغلاق الاتصال
        Timer::del($connection->auth_timer_id);
        break;
         ... تم تجاهله
    }
    ... تم تجاهله
}
```
