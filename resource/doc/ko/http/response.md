워커맨 4.x 버전부터 HTTP 서비스를 강화시켰습니다. 요청 클래스, 응답 클래스, 세션 클래스 및 [SSE](SSE.md)를 도입했습니다. 따라서 워커맨의 HTTP 서비스를 사용하려면 워커맨 4.x 이상의 버전을 사용하는 것을 강력히 권장합니다.

**아래 내용은 모두 워커맨 4.x 버전의 사용법이며, 워커맨 3.x와 호환되지 않습니다.**


## 참고 사항

 - 청크 또는 SSE 응답을 보내는 경우를 제외하고는 한 요청 안에서 여러 번의 응답을 보내는 것이 허용되지 않습니다. 즉, 한 요청 안에서 `$connection->send()`를 여러 번 호출하는 것은 허용되지 않습니다.
 - 모든 요청은 최종적으로 한 번의 `$connection->send()` 호출을 통해 응답을 전송해야 하며, 그렇지 않을 경우 클라이언트는 계속 대기하게 됩니다.

## 빠른 응답
HTTP 상태 코드를 변경할 필요가 없거나, 사용자 지정 헤더 및 쿠키를 사용하지 않아도 되는 경우에는 클라이언트에게 문자열을 직접 보내어 응답을 완료할 수 있습니다.

**예시**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // 클라이언트에게 'this is body'를 직접 전송
    $connection->send("this is body");
};

// 워커 실행
Worker::runAll();
```

## 상태 코드 변경
사용자 정의 상태 코드, 헤더 및 쿠키를 사용해야 하는 경우 `Workerman\Protocols\Http\Response` 응답 클래스를 사용해야 합니다. 예를 들어, `/404` 경로에 접근할 때 404 상태 코드와 `<h1>Sorry, the file does not exist</h1>` 내용을 반환하는 아래 예시를 살펴보겠습니다.

**예시**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    if ($request->path() === '/404') {
        $connection->send(new Response(404, [], '<h1>Sorry, the file does not exist</h1>'));
    } else {
        $connection->send('this is body');
    }
};

// 워커 실행
Worker::runAll();
```
`Response` 클래스가 이미 초기화된 후 상태 코드를 변경하려면 다음 메서드를 사용하세요.
```php
$response = new Response(200);
$response->withStatus(404);
$connection->send($response);
```

## 헤더 전송
마찬가지로, 헤더를 전송하려면 `Workerman\Protocols\Http\Response` 응답 클래스를 사용해야 합니다.

**예시**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $response = new Response(200, [
        'Content-Type' => 'text/html',
        'X-Header-One' => 'Header Value'
    ], 'this is body');
    $connection->send($response);
};

// 워커 실행
Worker::runAll();
```
응답 클래스가 이미 초기화된 후 헤더를 추가하거나 변경하려면 다음 메서드를 사용하세요.
```php
$response = new Response(200);
// 헤더 추가 또는 변경
$response->header('Content-Type', 'text/html');
// 여러 개의 헤더 추가 또는 변경
$response->withHeaders([
    'Content-Type' => 'application/ json',
    'X-Header-One' => 'Header Value'
]);
$connection->send($response);
```

## 리디렉션
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)

$worker = new Worker('http://0.0.0.0:8080');
$worker->onMessage = function($connection, $request)
{
    $location = '/test_location';
    $response = new Response(302, ['Location' => $location]);
    $connection->send($response);
};
Worker::runAll();
```

## 쿠키 전송
마찬가지로, 쿠키를 전송하려면 `Workerman\Protocols\Http\Response` 응답 클래스를 사용해야 합니다.

**예시**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $response = new Response(200, [], 'this is body');
    $response->cookie('name', 'tom');
    $connection->send($response);
};

// 워커 실행
Worker::runAll();
```

## 파일 전송
마찬가지로, 파일을 전송하려면 `Workerman\Protocols\Http\Response` 응답 클래스를 사용해야 합니다.

파일을 전송할 때는 다음과 같이 사용합니다.
```php
$response = (new Response())->withFile($file);
$connection->send($response);
```
 - workerman은 대용량 파일 전송을 지원합니다.
 - 대용량 파일(2M 이상)의 경우 workerman은 전체 파일을 한 번에 메모리에 읽지 않고 적절한 타이밍에 파일을 분할하여 송신합니다.
 - workerman은 클라이언트의 수신 속도에 따라 파일 읽기 및 전송 속도를 최적화하여 최대한 빠르게 파일을 전송하면서 메모리 사용을 최소화합니다.
 - 데이터 송신은 블로킹되지 않으므로 다른 요청 처리에 영향을 주지 않습니다.
 - 파일 전송 시, 다음 요청을 위해 서버가 수정된 경우 `Last-Modified` 헤더가 자동으로 추가되어 파일 전송을 최적화하여 성능을 향상시킵니다.
 - 전송되는 파일은 적절한 `Content-Type` 헤더를 자동으로 사용하여 브라우저로 전송됩니다.
 - 파일이 없는 경우 자동으로 404 응답으로 변환됩니다.

**예시**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $file = '/your/path/of/file';
    // if-modified-since 헤더를 확인하여 파일이 수정되었는지 확인
    if (!empty($if_modified_since = $request->header('if-modified-since'))) {
        $modified_time = date('D, d M Y H:i:s',  filemtime($file)) . ' ' . \date_default_timezone_get();
        // 파일이 수정되지 않았다면 304를 반환
        if ($modified_time === $if_modified_since) {
            $connection->send(new Response(304));
            return;
        }
    }
    // 파일이 수정되었거나 if-modified-since 헤더가 없는 경우 파일을 전송
    $response = (new Response())->withFile($file);
    $connection->send($response);
};

// 워커 실행
Worker::runAll();
```

## HTTP chunk 데이터 전송
 - 클라이언트에게 `Transfer-Encoding: chunked` 헤더를 가진 Response 응답을 먼저 보내야 합니다.
 - 이후의 chunk 데이터 전송은 `Workerman\Protocols\Http\Chunk` 클래스를 사용합니다.
 - 마지막에는 빈 chunk를 보내어 응답을 종료해야 합니다.

**예시**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
use Workerman\Protocols\Http\Chunk;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // 먼저 Transfer-Encoding: chunked 헤더를 가진 Response 응답을 보냅니다.
    $connection->send(new Response(200, array('Transfer-Encoding' => 'chunked'), 'hello'));
    // 이후 Chunk 데이터는 Workerman\Protocols\Http\Chunk 클래스를 사용하여 전송합니다.
    $connection->send(new Chunk('첫 번째 데이터'));
    $connection->send(new Chunk('두 번째 데이터'));
    $connection->send(new Chunk('세 번째 데이터'));
   //  마지막으로 빈 chunk를 보내어 응답을 종료합니다.
    $connection->send(new Chunk(''));
};

// 워커 실행
Worker::runAll();
```
