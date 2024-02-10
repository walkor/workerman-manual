# workerman/http-client
## 설명
[workerman/http-client](https://github.com/walkor/http-client)는 비동기 HTTP 클라이언트 컴포넌트입니다. 모든 요청과 응답은 비동기 및 논블로킹으로 처리되며 내장된 연결 풀과 메시지 요청 및 응답은 PSR7 규격을 준수합니다.

## 설치:
```php
composer require workerman/http-client
```

## 예제:

**get post request 사용법**

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

**파일 업로드**

```php
<?php
use Workerman\Worker;

require_once 'vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function () {
    $http = new Workerman\Http\Client();
    // 파일 업로드
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

**progress 스트림식 반환**

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

## 옵션
```php
<?php
require __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
$worker = new Worker();
$worker->onWorkerStart = function(){
    $options = [
        'max_conn_per_addr' => 128, // 도메인당 최대 동시 연결 수
        'keepalive_timeout' => 15,  // 통신이 없는 경우 연결을 닫을 시간
        'connect_timeout'   => 30,  // 연결 시간 초과
        'timeout'           => 30,  // 요청 보낸 후 응답을 기다리는 시간 초과
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

## 코루틴 사용법

> **주의**
> 코루틴 사용법은 workerman>=5.0, workerman/http-client>=2.0.0 및 composer require revolt/event-loop ^1.0.0 이 설치되어 있어야 합니다.

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

콜백 함수를 설정하지 않으면 클라이언트는 비동기 요청 결과를 동기적으로 반환하며 요청 프로세스는 현재 프로세스를 차단하지 않습니다. 즉, 동시에 요청을 처리할 수 있습니다.

## 주의:

1. 프로젝트는 먼저 `require __DIR__ . '/vendor/autoload.php';`를 로드해야 합니다.

2. 모든 비동기 코드는 workerman이 시작된 후의 실행 환경에서만 실행될 수 있습니다.

3. GatewayWorker, PHPSocket.io 등 workerman 기반의 모든 프로젝트를 지원합니다.
