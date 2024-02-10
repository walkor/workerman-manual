# คุณลักษณะ transport

```(ต้องการ workerman >= 3.3.4)```

กำหนดค่า transport ซึ่งสามารถเป็นค่า [tcp](https://baike.baidu.com/subview/32754/8048820.htm) และ [ssl](https://baike.baidu.com/view/525499.htm) โดยค่าเริ่มต้นคือ tcp

เมื่อ transport เป็น [ssl](https://baike.baidu.com/view/525499.htm) จะต้องมีการติดตั้ง [openssl extension](https://php.net/manual/zh/book.openssl.php) สำหรับ PHP

เมื่อใช้ Workerman เป็น client ที่เชื่อมต่อกับ server ผ่านการเข้ารหัสด้วย ssl (เช่นการเชื่อมต่อ https, wss) โปรดกำหนดค่าตัวเลือกนี้เป็น ```ssl``` เช่นตัวอย่างด้านล่าง

### ตัวอย่าง (การเชื่อมต่อ https)
```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function($task)
{
    $connection_to_baidu = new AsyncTcpConnection('tcp://www.baidu.com:443');

    // กำหนดให้เป็นการเชื่อมต่อที่เข้ารหัสด้วย ssl
    $connection_to_baidu->transport = 'ssl';

    $connection_to_baidu->onConnect = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "เชื่อมต่อสำเร็จ\n";
        $connection_to_baidu->send("GET / HTTP/1.1\r\nHost: www.baidu.com\r\nConnection: keep-alive\r\n\r\n");
    };
    $connection_to_baidu->onMessage = function(AsyncTcpConnection $connection_to_baidu, $http_buffer)
    {
        echo $http_buffer;
    };
    $connection_to_baidu->onClose = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "เชื่อมต่อถูกปิด\n";
    };
    $connection_to_baidu->onError = function(AsyncTcpConnection $connection_to_baidu, $code, $msg)
    {
        echo "รหัสข้อผิดพลาด:$code ข้อความ:$msg\n";
    };
    $connection_to_baidu->connect();
};

Worker::runAll();
```
