# workerman/http-client
## คำอธิบาย
[workerman/http-client](https://github.com/walkor/http-client) เป็นคอมโพเนนต์ HTTP แบบอะซิงโครนัส ทั้งคำขอและการตอบคำขอเป็นแบบอะซิงโครนัสทั้งหมด มีพูลการเชื่อมต่อภายใน และข้อความคำขอและการตอบคำขอเป็นไปตามมาตรฐาน PSR7

## การติดตั้ง:
```composer require workerman/http-client```

## ตัวอย่าง:

**การใช้ get post request**

```php
use Workerman\Worker;

require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function () {
    $http = new Workerman\Http\Client();

    $http->get('https://example.com/', function ($response) {
        var_dump($response->getStatusCode());
        echo $response->getBody();
    }, function ($exception) {
        echo $exception;
    });

    $http->post('https://example.com/', ['key1' => 'value1', 'key2' => 'value2'], function ($response) {
        var_dump($response->getStatusCode());
        echo $response->getBody();
    }, function ($exception) {
        echo $exception;
    });

    $http->request('https://example.com/', [
        'method' => 'POST',
        'version' => '1.1',
        'headers' => ['Connection' => 'keep-alive'],
        'data' => ['key1' => 'value1', 'key2' => 'value2'],
        'success' => function ($response) {
            echo $response->getBody();
        },
        'error' => function ($exception) {
            echo $exception;
        }
    ]);
};
Worker::runAll();
```

**การอัปโหลดไฟล์**

```php
<?php
use Workerman\Worker;

require_once 'vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function () {
    $http = new Workerman\Http\Client();
    // อัปโหลดไฟล์
    $multipart = new \Workerman\Psr7\MultipartStream([
        [
            'name' => 'file',
            'contents' => fopen(__FILE__, 'r')
        ],
        [
            'name' => 'json',
            'contents' => json_encode(['a'=>1, 'b'=>2])
        ]
    ]);
    $boundary = $multipart->getBoundary();
    $http->request('http://127.0.0.1:8787', [
        'method' => 'POST',
        'version' => '1.1',
        'headers' => ['Connection' => 'keep-alive', 'Content-Type' => "multipart/form-data; boundary=$boundary"],
        'data' => $multipart,
        'success' => function ($response) {
            echo $response->getBody();
        },
        'error' => function ($exception) {
            echo $exception;
        }
    ]);
};

Worker::runAll();
```

**การสตรีมข้อมูลที่กำลังให้คำตอบ**

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Connection\TcpConnection;
use Workerman\Http\Client;
use Workerman\Protocols\Http\Chunk;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
use Workerman\Worker;

$worker = new Worker('http://0.0.0.0:1234');
$worker->onMessage = function (TcpConnection $connection, Request $request) {
    $http = new Client();
    $http->request('https://api.bla.cn/v1/chat/completions', [
        'method' => 'POST',
        'data' => json_encode([
            'model' => 'gpt-3.5-turbo',
            'temperature' => 1,
            'stream' => true,
            'messages' => [['role' => 'user', 'content' => 'hello']],
        ]),
        'headers' => [
            'Content-Type' => 'application/json',
            'Authorization' => 'Bearer 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',
        ],
        'progress' => function($buffer) use ($connection) {
            $connection->send(new Chunk($buffer));
        },
        'success' => function($response) use ($connection) {
            $connection->send(new Chunk(''));
        },
    ]);
    $connection->send(new Response(200, [
        //"Content-Type" => "application/octet-stream",
        "Transfer-Encoding" => "chunked",
    ], ' '));
};
Worker::runAll();
```

## ตัวเลือก
```php
<?php
require __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
$worker = new Worker();
$worker->onWorkerStart = function(){
    $options = [
        'max_conn_per_addr' => 128, // จำนวนการเชื่อมต่อพร้อมกันสูงสุดต่อโดเมน
        'keepalive_timeout' => 15,  // เวลาที่จะปิดการเชื่อมต่อหากไม่มีการสื่อสาร
        'connect_timeout'   => 30,  // เวลาที่ใช้ในการเชื่อมต่อ
        'timeout'           => 30,  // เวลาที่รอการตอบสนองหลังจากส่งคำขอ
    ];
    $http = new Workerman\Http\Client($options);

    $http->get('http://example.com/', function($response){
        var_dump($response->getStatusCode());
        echo $response->getBody();
    }, function($exception){
        echo $exception;
    });
};
Worker::runAll();
```
## การใช้งานกอริทึม

> **โปรดทราบ**
> การใช้งานกอริทึม ต้องการ Workerman เวอร์ชั่น 5.0 ขึ้นไป รวมถึง workerman/http-client เวอร์ชั่น 2.0.0 ขึ้นไป และติดตั้ง composer require revolt/event-loop ^1.0.0

```php
use Workerman\Worker;

require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function () {
    $http = new Workerman\Http\Client();

    $response = $http->get('https://example.com/');
    var_dump($response->getStatusCode());
    echo $response->getBody();

    $response = $http->post('https://example.com/', ['key1' => 'value1', 'key2' => 'value2']);
    var_dump($response->getStatusCode());
    echo $response->getBody();
    

    $response = $http->request('https://example.com/', [
        'method' => 'POST',
        'version' => '1.1',
        'headers' => ['Connection' => 'keep-alive'],
        'data' => ['key1' => 'value1', 'key2' => 'value2'],
    ]);
    echo $response->getBody();
};
Worker::runAll();
```

เมื่อไม่ได้กำหนดฟังก์ชั่น callback ไว้ เซิร์ฟเวอร์จะรับข้อมูลของคำขอ แบบไม่สมุนคลาสให้กับผลลัพธ์สำหรับคำขอแบบไม่สมุน กระบวนการคำขอไม่ block กระบวนการปัจจุบัน ก็คือสามารถทำคำขอพร้อมกันได้

## ข้อควรระวัง：

1. โปรเจ็คควรโหลด `require __DIR__ . '/vendor/autoload.php';` ก่อนที่จะรัน

2. โค้ดผลลัพธ์ของกระบวนการคำขอทั้งหมดจะต้องรันใน workerman environment หลังจากที่ workerman ได้เริ่มการทำงาน

3. รองรับโปรเจ็คที่พัฒนาขึ้นบน workerman ทุกอย่างทุกอย่าง ซึ่งรวมถึง GatewayWorker, PHPSocket.io ฯลฯ
