# workerman/http-client
## Giới thiệu
[workerman/http-client](https://github.com/walkor/http-client) là một thành phần HTTP client bất đồng bộ. Tất cả các yêu cầu và phản hồi là bất đồng bộ và không chặn, có tích hợp sẵn bể kết nối, yêu cầu và phản hồi tuân theo chuẩn PSR7.

## Cài đặt:
```composer require workerman/http-client```

## Ví dụ:

**Get và Post request**

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

**Tải lên tập tin**

```php
<?php
use Workerman\Worker;

require_once 'vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function () {
    $http = new Workerman\Http\Client();
    // Tải lên tập tin
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

**Phản hồi tiến trình dữ liệu**

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

## Tùy chọn

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
$worker = new Worker();
$worker->onWorkerStart = function(){
    $options = [
        'max_conn_per_addr' => 128, // Số lượng kết nối tối đa mỗi địa chỉ
        'keepalive_timeout' => 15,  // Thời gian kết nối không giao tiếp sẽ đóng
        'connect_timeout'   => 30,  // Thời gian kết nối vượt quá sẽ hết thời gian
        'timeout'           => 30,  // Thời gian chờ phản hồi sau khi yêu cầu
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

## Sử dụng coroutine

> **Note**
> Sử dụng coroutine yêu cầu workerman>=5.0, workerman/http-client>=2.0.0 và cài đặt composer require revolt/event-loop ^1.0.0

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

Khi không thiết lập hàm callback, client sẽ trả về kết quả yêu cầu bất đồng bộ theo cách đồng bộ, quá trình yêu cầu không chặn quá trình hiện tại, có nghĩa là có thể xử lý đồng thời.

## Lưu ý:

1. Dự án cần tải lên `require __DIR__ . '/vendor/autoload.php';` đầu tiên.

2. Tất cả các mã bất đồng bộ chỉ có thể chạy trong môi trường chạy sau khi workerman đã được khởi động.

3. Hỗ trợ tất cả các dự án dựa trên workerman, bao gồm GatewayWorker, PHPSocket.io, vv.
