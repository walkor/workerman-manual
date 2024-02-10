# 설명

workerman은 4.x 버전부터 HTTP 서비스를 강화했습니다. 요청 클래스, 응답 클래스, 세션 클래스 및 [SSE](SSE.md)를 도입했습니다. workerman의 HTTP 서비스를 사용하려면 workerman 4.x 이상의 최신 버전을 사용하는 것을 강력히 권장합니다.

**다음은 workerman 4.x 버전용 사용법이며, workerman 3.x와 호환되지 않습니다.**


# 세션 객체 얻기
```php
$session = $request->session();
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
    $session = $request->session();
    $session->set('name', 'tome');
    $connection->send($session->get('name'));
};

// 워커 실행
Worker::runAll();
```
**주의사항**
- 세션은 `$connection->send()` 호출 전에 작동해아합니다.
- 세션이 객체 소멸 시에 자동으로 변경 사항을 저장하므로, `$request->session()`에서 반환된 객체를 전역 배열이나 클래스 멤버에 저장하여 세션을 저장할 수 없습니다.
- 세션 기본적으로 디스크 파일에 저장되며, 더 나은 성능을 원한다면 Redis를 사용하는 것을 권장합니다.


## 모든 세션 데이터 가져오기
```php
$session = $request->session();
$all = $session->all();
```
반환되는 것은 배열입니다. 세션 데이터가 없는 경우 빈 배열이 반환됩니다.

**예시**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;

require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('name', 'tom');
    $connection->send(var_export($session->all(), true));
};

// 워커 실행
Worker::runAll();
```


## 세션에서 특정 값 가져오기
```php
$session = $request->session();
$name = $session->get('name');
```
데이터가 존재하지 않는 경우에는 null이 반환됩니다.

또한 get 메서드에 두 번째 인수로 기본값을 전달하여 세션 배열에서 해당 값이 없는 경우 기본값을 반환할 수 있습니다. 예를 들어:
```php
$session = $request->session();
$name = $session->get('name', 'tom');
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
    $session = $request->session();
    $connection->send($session->get('name', 'tom'));
};

// 워커 실행
Worker::runAll();
```


## 세션 저장
특정 데이터를 저장할 때는 set 메소드를 사용합니다.
```php
$session = $request->session();
$session->set('name', 'tom');
```
set은 반환 값을 가지지 않으며, 세션 객체가 소멸될 때 세션은 자동으로 저장됩니다.

여러 값을 저장할 때는 put 메소드를 사용합니다.
```php
$session = $request->session();
$session->put(['name' => 'tom', 'age' => 12]);
```
마찬가지로 put은 반환 값을 가지지 않습니다.

**예시**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;

require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('name', 'tom');
    $connection->send($session->get('name'));
};

// 워커 실행
Worker::runAll();
```


## 세션 데이터 삭제
특정한 하나 또는 여러개의 세션 데이터를 삭제하려면 `forget` 메소드를 사용합니다.
```php
$session = $request->session();
// 항목 하나 삭제
$session->forget('name');
// 다수 항목 삭제
$session->forget(['name', 'age']);
```

또한 시스템에서는 forget 메소드와는 달리 하나의 항목만 삭제할 수 있는 delete 메소드를 제공합니다.
```php
$session = $request->session();
// $session->forget('name');와 동일합니다
$session->delete('name');
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
    $request->session()->forget('name');
    $connection->send('ok');
};

// 워커 실행
Worker::runAll();
```


## 세션의 특정 값 가져와 삭제하기
```php
$session = $request->session();
$name = $session->pull('name');
```
아래 코드와 동일한 효과가 있습니다.
```php
$session = $request->session();
$value = $session->get($name);
$session->delete($name);
```
해당 세션이 존재하지 않는 경우 null이 반환됩니다.

**예시**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;

require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->session()->pull('name'));
};

// 워커 실행
Worker::runAll();
```


## 모든 세션 데이터 삭제
```php
$request->session()->flush();
```
반환 값이 없으며, 세션 객체가 소멸될 때 저장소에서 세션은 자동으로 제거됩니다.

**예시**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;

require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $request->session()->flush();
    $connection->send('ok');
};

// 워커 실행
Worker::runAll();
```


## 해당 세션 데이터의 존재 여부 확인
```php
$session = $request->session();
$has = $session->has('name');
```
해당 세션이 존재하지 않거나 세션 값이 null인 경우에는 false를 반환하고, 그렇지 않은 경우에는 true를 반환합니다.

```php
$session = $request->session();
$has = $session->exists('name');
```
위 코드도 세션 데이터의 존재 여부를 확인하는 데 사용되며, 차이점은 해당 세션 항목 값이 null인 경우에도 true를 반환합니다.
