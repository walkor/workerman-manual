**كيفية إرسال واستقبال البيانات الست عشرية**

**استقبال البيانات الست عشرية**

بمجرد استلام البيانات، يمكن استخدام الدالة `bin2hex($data)` لتحويل البيانات إلى نظام الست عشري.

**إرسال البيانات الست عشرية**

قبل إرسال البيانات، يجب استخدام `hex2bin($data)` لتحويل البيانات الست عشرية إلى بيانات ثنائية للإرسال.

**مثال:**

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:8080');
$worker->onMessage = function(TcpConnection $connection, $data){
    // الحصول على البيانات الست عشرية
    $hex_data = bin2hex($data);
    // إرسال البيانات الست عشرية إلى العميل
    $connection->send(hex2bin($hex_data));
};
Worker::runAll();
```
