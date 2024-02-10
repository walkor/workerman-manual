# เมธอด __construct
```php
void AsyncTcpConnection::__construct(string $remote_address, $context_option = null)
```
สร้างอ็อบเจ็กต์การเชื่อมต่อแบบไม่สามารถทำงานพร้อมกันได้

AsyncTcpConnection สามารถทำให้ Workerman ทำหน้าที่เป็นผู้ใช้เชื่อมต่อไปที่เซิร์ฟเวอร์ระยะไกลแบบไม่สามารถทำงานพร้อมกันได้ และใช้ send interface และ onMessage callback เพื่อทำการส่งและจัดการข้อมูลที่เชื่อมต่อแบบไม่สามารถทำงานพร้อมกัน

## อาร์กิวเม้นต์
Parameter: ``` remote_address ```

ที่อยู่ที่เชื่อมต่อ เช่น 
 ``` tcp://www.baidu.com:80 ```
 ``` ssl://www.baidu.com:443 ```
 ``` ws://echo.websocket.org:80 ```
 ``` frame://192.168.1.1:8080 ```
 ``` text://192.168.1.1:8080 ```

Parameter: ``` $context_option ```

 ```ต้องใช้พารามิเตอร์นี้ (workerman >= 3.3.5)```

ใช้เพื่อตั้งค่า socket context เช่น การใช้ ```bindto``` ตั้งค่าให้ใช้ (เครือข่ายการ์ด) ip และพอร์ตเมื่อต้องการเข้าถึงเครือข่ายภายนอก การตั้งค่า SSL ฯลฯ

สามารถอ้างอิงได้ที่ [stream_context_create](https://php.net/manual/en/function.stream-context-create.php) และ [socket context options](https://php.net/manual/zh/context.socket.php) และ [SSL context options](https://php.net/manual/zh/context.ssl.php) 

## คำแนะนำ
ในปัจจุบัน AsyncTcpConnection รองรับโปรโตคอล [tcp](https://baike.baidu.com/subview/32754/8048820.htm) และ [ssl](https://baike.baidu.com/view/525499.htm) และ [ws](appendices/about-ws.md) และ [frame](appendices/about-frame.md) และ [text](appendices/about-text.md)

รองรับโปรโตคอลที่กำหนดเอง โปรดดูที่ [วิธีกำหนดโปรโตคอลเอง](../protocols/how-protocols.md)

โปรโตคอล [ssl](https://baike.baidu.com/view/525499.htm) ต้องใช้ Workerman >= 3.3.4 และติดตั้ง [openssl extension](https://php.net/manual/zh/book.openssl.php) 

ปัจจุบันไม่รองรับโปรโตคอล [http](https://baike.baidu.com/view/9472.htm) ของ AsyncTcpConnection

สามารถใช้ ```new AsyncTcpConnection('ws://...')``` เหมือนเป็นเบราว์เซอร์เพื่อสร้างการเชื่อมต่อ websocket ไปยังเซิร์ฟเวอร์ websocket ระยะไกลใน Workerman ดูตัวอย่างได้ที่ [ตัวอย่าง](../appendices/about-ws.md) แต่ไม่สามารถใช้ ```new AsyncTcpConnection('websocket://...')``` เพื่อสร้างการเชื่อมต่อ websocket ใน Workerman ได้
## ตัวอย่าง

### ตัวอย่าง 1: เข้าถึงบริการ http ภายนอกโดยแบบไม่สะดุดตา
```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// เมื่อเริ่มกระบวนการ จะส่งการเชื่อมต่อแบบไม่สะดุดตาไปยัง www.baidu.com และส่งข้อมูลเพื่อรับข้อมูล
$task->onWorkerStart = function($task)
{
    // ไม่ support http โดยตรง แต่สามารถใช้ tcp จำลองการส่งข้อมูลโปรโตคอล http
    $connection_to_baidu = new AsyncTcpConnection('tcp://www.baidu.com:80');
    // เมื่อเชื่อมต่อสำเร็จ ส่งข้อมูลคำขอ http
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
        echo "การเชื่อมต่อปิด\n";
    };
    $connection_to_baidu->onError = function(AsyncTcpConnection $connection_to_baidu, $code, $msg)
    {
        echo "รหัสข้อผิดพลาด: $code ข้อความ: $msg\n";
    };
    $connection_to_baidu->connect();
};

// รัน worker
Worker::runAll();
```

### ตัวอย่าง 2: เข้าถึงบริการ websocket ภายนอกโดยแบบไม่สะดุดตา และกำหนดไอพีและพอร์ตโดยใช้
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // กำหนดไอพีและพอร์ตของโฮสต์เราที่จะเข้าถึง (แต่ละการเชื่อมต่อ socket จะใช้พอร์ตโฮสต์เราหนึ่ง)
    $context_option = array(
        'socket' => array(
            // ไอพี ต้องเป็นไอพีของการ์ดเน็ตเวิร์กของเครื่องเรา และสามารถเข้าถึงโฮสต์เป้าหมายได้มิฉะนั้นไม่สามารถใช้งาน
            'bindto' => '114.215.84.87:2333',
        ),
    );

    $con = new AsyncTcpConnection('ws://echo.websocket.org:80', $context_option);

    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```


### ตัวอย่าง 3: เข้าถึงพอร์ต wss ภายนอกโดยแบบไม่สะดุดตา และกำหนดใบรับรอง ssl ภายใน
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // ระบุไอพีและพอร์ตของโฮสต์เป้าหมายและใบรับรอง ssl ภายใน
    $context_option = array(
        'socket' => array(
            // ไอพี ต้องเป็นไอพีของการ์ดเน็ตเวิร์กของเครื่องเรา และสามารถเข้าถึงโฮสต์เป้าหมายได้มิฉะนั้นไม่สามารถใช้งาน
            'bindto' => '114.215.84.87:2333',
        ),
        // ตัวเลือก ssl โปรดดูที่ https://php.net/manual/zh/context.ssl.php
        'ssl' => array(
            // เส้นทางของใบรับรองท้องถิ่น ต้องเป็นรูปแบบ PEM และประกอบด้วยใบรับรองท้องถิ่นและกุญแจส่วนตัวของเรา
            'local_cert'        => '/your/path/to/pemfile',
            // รหัสผ่านของไฟล์ local_cert
            'passphrase'        => 'your_pem_passphrase',
            // อนุญาตให้ใช้ใบรับรองสำหรับตนเองหรือไม่
            'allow_self_signed' => true,
            // ต้องการตรวจสอบใบรับรอง SSL หรือไม่
            'verify_peer'       => false
        )
    );

    // เริ่มการเชื่อต่อแบบไม่สะดุดตา
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443', $context_option);

    // กำหนดวิธีการเข้าถึงแบบ ssl
    $con->transport = 'ssl';

    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```
