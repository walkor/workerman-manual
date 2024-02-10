# 설명
workerman은 4.x 버전부터 HTTP 서비스를 강화하여 지원합니다. 요청 클래스, 응답 클래스, 세션 클래스 및 [SSE](SSE.md)를 도입했습니다. Workerman의 HTTP 서비스를 사용하려는 경우, workerman 4.x 이상의 높은 버전을 강력히 권장합니다.

**아래 내용은 모두 workerman 4.x 버전의 사용법으로 workerman 3.x와 호환되지 않습니다.**

## 요청 객체 가져오기
요청 객체는 onMessage 콜백 함수 내에서 항상 가져옵니다. 프레임워크는 Request 객체를 자동으로 콜백 함수의 두 번째 매개변수를 통해 전달합니다.

**예시**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // $request는 요청 객체이며, 여기서는 요청 객체를 조작하지 않고 브라우저에게 hello를 반환합니다.
    $connection->send("hello");
};

// worker 실행
Worker::runAll();
```

웹 브라우저가 `http://127.0.0.1:8080`을 방문하면 `hello`가 반환됩니다.

## GET 요청 매개변수 가져오기

**전체 GET 배열 가져오기**
```php
$get = $request->get();
```
요청에 GET 매개변수가 없으면 빈 배열이 반환됩니다.

**GET 배열의 특정 값 가져오기**
```php
$name = $request->get('name');
```
GET 배열에 해당 값이 없으면 null이 반환됩니다.

또한 get 메서드에 기본값을 두 번째 매개변수로 전달할 수도 있습니다. GET 배열에서 해당 값이 없는 경우 기본값을 반환합니다. 예:
```php
$name = $request->get('name', 'tom');
```

**예시**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->get('name'));
};

// worker 실행
Worker::runAll();
```

웹 브라우저가 `http://127.0.0.1:8080?name=jerry&age=12`을 방문하면 `jerry`가 반환됩니다.

(계속합니다...)
## 요청 메서드 가져오기
```php
$method = $request->method();
```
반환 값은 `GET`, `POST`, `PUT`, `DELETE`, `OPTIONS`, `HEAD` 중 하나일 수 있습니다.

**예시**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request) {
    $connection->send($request->method());
};

// worker 실행
Worker::runAll();
```

## 요청 URI 가져오기
```php
$uri = $request->uri();
```
요청된 URI를 반환하며, path 및 queryString 부분을 포함합니다.

**예시**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request) {
    $connection->send($request->uri());
};

// worker 실행
Worker::runAll();
```
웹 브라우저가 `http://127.0.0.1:8080/user/get.php?uid=10&type=2`에 액세스하면 `/user/get.php?uid=10&type=2`를 반환합니다.

## 요청 경로 가져오기
```php
$path = $request->path();
```
요청된 경로 부분을 반환합니다.

**예시**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request) {
    $connection->send($request->path());
};

// worker 실행
Worker::runAll();
```
웹 브라우저가 `http://127.0.0.1:8080/user/get.php?uid=10&type=2`에 액세스하면 `/user/get.php`를 반환합니다.

## 요청 쿼리스트링 가져오기
```php
$query_string = $request->queryString();
```
요청된 쿼리스트링 부분을 반환합니다.

**예시**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request) {
    $connection->send($request->queryString());
};

// worker 실행
Worker::runAll();
```
웹 브라우저가 `http://127.0.0.1:8080/user/get.php?uid=10&type=2`에 액세스하면 `uid=10&type=2`를 반환합니다.

## 요청 HTTP 버전 얻기
```php
$version = $request->protocolVersion();
```
문자열 `1.1` 또는 `1.0`을 반환합니다.

**예시**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request) {
    $connection->send($request->protocolVersion());
};

// worker 실행
Worker::runAll();
```

## 요청 세션 ID 얻기
```php
$sid = $request->sessionId();
```
글자와 숫자로 구성된 문자열을 반환합니다.

**예시**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request) {
    $connection->send($request->sessionId());
};

// worker 실행
Worker::runAll();
```
