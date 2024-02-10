# Fiber 코루틴
workerman은 5.0.0부터 [Fiber 코루틴(섬)](https://www.php.net/manual/zh/language.fibers.php)을 지원합니다.

> **주의**
> Fiber 기능을 사용하려면 PHP >= 8.1이어야 하며 `composer require revolt/event-loop ^1.0.0`을 설치해야 합니다.

### 소개
Fiber는 PHP에 내장된 코루틴(섬)으로, PHP 코드를 중단시키고 필요한 시점에 다시 실행할 수 있게 합니다. 그 주요 기능은 개발자가 동기식으로 비동기 논블로킹 코드를 작성할 수 있게 하여 코드의 유지보수성을 크게 향상시키는 것입니다.

### 예시
아래는 코루틴과 비동기 콜백 프로그래밍의 차이를 예시로 비교한 것입니다. HTTP 인터페이스를 호출하고 1초 지연 응답이 필요한 요구사항이 있다고 가정했을 때, 비동기 콜백과 코루틴의 작성 방법은 각각 다음과 같습니다.

**비동기 콜백 방식**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Http\Client;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    static $http;
    $http = $http ?: new Client();
    // HTTP 인터페이스 요청
    $http->get('http://example.com/', function ($response) use ($connection) {
        // 1초 지연 후 전송
        Timer::add(1, function() use ($connection, $response) {
            // 브라우저로 데이터 전송
            $connection->send((string)$response->getBody());
        }, null, false);
    });
};

Worker::runAll();
```

**코루틴 방식**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Http\Client;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    static $http;
    $http = $http ?: new Client();
    // HTTP 인터페이스 호출
    $response = $http->get('http://example.com/');
    // 1초 지연
    Timer::sleep(1);
    // 데이터 전송
    $connection->send((string)$response->getBody());
};

Worker::runAll();
```

> **주의**
> 위 코드는 `composer require workerman/http-client ^2.0.0`을 설치해야 합니다.

이 두 가지 방식은 모두 비동기 논블로킹으로 실행되며 효율적으로 동작하지만, 코루틴 방식은 비동기 콜백보다 읽기 쉽고 유지보수가 더 쉬워집니다.

### Fiber 주의사항
* Fiber 코루틴은 Pdo, Redis 및 PHP 내부 차단 함수를 코루틴화할 수 없으며, 이러한 확장 및 함수를 사용하는 것은 여전히 차단 호출입니다.
* 현재 사용 가능한 Fiber 코루틴 클라이언트는 [workerman/http-client](../components/workerman-http-client.md), [workerman/redis](../components/workerman-redis.md)가 있습니다.

# Swoole 코루틴
workerman v5는 Swoole을 이벤트 드라이버로 사용하는 것을 동시에 지원합니다.

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Swoole\Coroutine\Http\Client;
use Swoole\Coroutine;

// 여기서 Swoole을 이벤트 드라이버로 수동 설정해야 합니다.
Worker::$eventLoopClass = Workerman\Events\Swoole::class;
$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    Coroutine::create(function() use ($connection) {
        $cli = new Client('example.com', 80);
        $cli->get('/get');
        $connection->send($cli->body);
    });
};

Worker::runAll();
```

**팁**
* Swoole 5.0이나 그 이상을 사용하는 것이 권장됩니다.
* Workerman이 swoole의 코루틴을 지원하도록 Swoole을 이벤트 드라이버로 설정할 수 있습니다.
* Swoole을 이벤트 드라이버로 사용할 때, event 확장을 설치할 필요가 없습니다.
* Swoole은 기본적으로 일관된 코루틴을 활성화하지 않으며, 이는 Pdo, Redis, PHP 내장 파일 읽기/쓰기에 기반한 차단 호출임을 의미합니다.
* 일관된 코루틴을 활성화하려면 수동으로 `\Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_ALL);`를 호출해야 합니다.

자세한 내용은 [Swoole 매뉴얼](https://wiki.swoole.com/)을 참조하십시오.

더 많은 정보는 [이벤트 드라이브](appendices/event.md)를 참조하십시오.

# 코루틴에 관하여
먼저, 코루틴에 지나치게 신뢰하지 않는 것이 좋습니다. 데이터베이스, Redis 등의 저장소가 내부 네트워크에 있을 때, 여러 프로세스의 차단 호출이 종종 코루틴보다 더 빠를 수 있습니다. [techempower.com의 3년간의 벤치마크 데이터](https://www.techempower.com/benchmarks/#section=data-r21&l=zik073-6bj&test=db)에서 확인되었듯이, workerman의 차단 호출 기능은 swoole의 데이터베이스 연결 풀과 코루틴보다 성능이 우수하며, 심지어 go 언어의 gin, echo 등의 코루틴 프레임워크보다 거의 2배 빠릅니다.

workerman은 이미 PHP 애플리케이션 성능을 몇 배에서 몇십 배로 향상시켰으며, 대부분의 workerman 프로젝트에 코루틴을 추가하더라도 성능 향상이 크지 않을 수 있습니다. 외부 HTTP 호출과 같이 느린 호출이 시스템에 있는 경우 코루틴을 사용하여 성능을 향상시킬 수 있습니다.
