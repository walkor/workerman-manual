```php
function reConnect(float $delay = 0)
```

``` (ต้องการ Workerman เวอร์ชัน >= 3.3.5) ```

การเชื่อมต่อใหม่ เรียกใช้จากคลาส `onClose` เพื่อทำการเชื่อมต่อใหม่หลังจากการตัดการเชื่อมต่อ

หากการเชื่อมต่อถูกตัดเชื่อมเพราะปัญหาเครือข่ายหรือบริการที่ตรงข้ามรีสตาร์ต ก็สามารถเรียกใช้เมธอดนี้เพื่อเชื่อมต่อใหม่ได้


### พารามิเตอร์
 ``` $delay ```

ระยะเวลาการล่าช้าก่อนที่จะเริ่มการเชื่อมต่อใหม่ ในหน่วยเป็นวินาที รองรับทศนิยม และสามารถระบุได้ถึงมิลลิวินาที

หากไม่ระบุค่าหรือค่าเป็น 0 จะหมายถึงเริ่มการเชื่อมต่อใหม่ทันที

การระบุพารามิเตอร์เพื่อล่าช้าการเชื่อมต่อใหม่จะทำให้หลีกเลี่ยงการใช้งาน CPU สูงเนื่องจากไม่สามารถเชื่อมต่อได้เนื่องจากปัญหาของบริการอีกเครื่อง


### ค่าส่งคืน
ไม่มีการคืนค่า

### ตัวอย่าง

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker)
{
    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };
    $con->onMessage = function(AsyncTcpConnection $con, $msg) {
        echo "recv $msg\n";
    };
    $con->onClose = function(AsyncTcpConnection $con) {
        // หากการเชื่อมต่อถูกตัด ให้ทำการเชื่อมต่อใหม่ในอีก 1 วินาที
        $con->reConnect(1);
    };
    $con->connect();
};

Worker::runAll();
```

> ** ข้อควรระวัง **
> เมื่อทำการเชื่อมต่อใหม่สำเร็จแล้ว เมธอด onConnect ของ $con จะถูกเรียกอีกครั้ง (หากถูกตั้งค่า) ในบางครั้งเราอาจต้องการให้เมธอด onConnect ทำงานเพียงครั้งเดียวเท่านั้น และไม่ต้องทำการเชื่อมต่อใหม่ ดูตัวอย่างด้านล่าง

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker) {
    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    $con->onConnect = function(AsyncTcpConnection $con) {
        static $is_first_connect = true;
        if (!$is_first_connect) return;
        $is_first_connect = false;
        $con->send('hello');
    };
    $con->onMessage = function(AsyncTcpConnection $con, $msg) {
        echo "recv $msg\n";
    };
    $con->onClose = function(AsyncTcpConnection $con) {
        // หากการเชื่อมต่อถูกตัด ให้ทำการเชื่อมต่อใหม่ในอีก 1 วินาที
        $con->reConnect(1);
    };
    $con->connect();
};

Worker::runAll();
```
