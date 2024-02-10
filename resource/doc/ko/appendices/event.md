Workerman 현재 지원하는 이벤트 드라이버는 다음과 같습니다.

| 이름  | 의존 확장 | 코루틴 지원 |  우선순위  |  workerman 버전  |
|-----|------|--|-----|
|  Workerman\Events\Select   |   없음   | 지원 안 함  |  커널 기본 지원   |  >=3.0  ｜
|  Workerman\Events\Revolt   |   event(선택 사항)   | 지원  |  [revolt/event-loop](https://github.com/revoltphp/event-loop) 설치 필요   |  >=5.0  |
|  Workerman\Events\Event   |   event   | 지원 안 함 |  커널 기본 지원   |  >=3.0  |
|  Workerman\Events\Swoole   |  [swoole](https://github.com/swoole/swoole-src)   | 지원  |  수동 설정 필요   |  >=4.0  |
|  Workerman\Events\Swow   |   [swow](https://github.com/swow/swow)   | 지원  |  수동 설정 필요   |  >=5.0  |

* 각각의 커널 드라이버는 별도의 기능을 제공합니다. 예를 들어 `Revolt` 사용하면 workerman이 PHP 내장 [Fiber 코루틴](https://www.php.net/manual/zh/language.fibers.php)을 지원하게 되고, `Swoole`을 사용하면 workerman이 Swoole의 코루틴을 지원하게 됩니다.
* 각각의 이벤트 드라이버는 상호 배타적인 관계에 있으며, 예를 들어 `Revolt`의 Fiber 코루틴을 사용할 때는 Swoole이나 Swow의 코루틴을 사용할 수 없습니다.
* `Revolt`은 `composer require revolt/event-loop ^1.0.0`를 설치해야 하며, 설치 후 workerman 코어에서 자동으로 우선적인 이벤트 드라이버로 설정됩니다.
* `Swoole`과 `Swow`는 `Worker::$eventLoopClass`를 수동으로 설정해야 합니다(아래 단락 참조).
* swoole은 기본적으로 [코루틴 Runtime을 자동으로 활성화](https://wiki.swoole.com/#/runtime?id=runtime)하지 않습니다. 이는 Pdo, Redis, PHP 내장 파일 읽기/쓰기가 여전히 차단 호출로 이뤄지는 것을 의미합니다.
* swoole의 코루틴을 자동으로 활성화하려면 `\Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_ALL);`을 수동으로 호출해야 합니다.

> **주의**
> swow 확장은 일부 PHP 내장 함수의 동작을 자동으로 변경하며, swow 확장을 활성화했지만 이벤트 드라이버로 swow를 사용하지 않을 경우에는 workerman이 요청 및 신호에 대응하지 못할 수 있습니다. 따라서 swow를 이벤트 드라이버로 사용하지 않을 경우 php.ini에서 swow를 주석 처리해야 합니다.

자세한 내용은 [workerman Coroutine](../fiber.md)을 참조하십시오.

# Workerman에 이벤트 드라이버 설정

아래는 Workerman에 수동으로 이벤트 드라이버를 설정하는 방법입니다.

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Swoole\Coroutine\Http\Client;
use Swoole\Coroutine;

// 이벤트 드라이버를 수동으로 설정
Worker::$eventLoopClass = Workerman\Events\Revolt::class;
//Worker::$eventLoopClass = Workerman\Events\Select::class;
//Worker::$eventLoopClass = Workerman\Events\Event::class;
//Worker::$eventLoopClass = Workerman\Events\Swoole::class;
//Worker::$eventLoopClass = Workerman\Events\Swow::class;
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
