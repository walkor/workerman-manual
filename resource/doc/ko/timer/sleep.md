# sleep
```php
int \Workerman\Timer::sleep(float $delay)
```
PHP 내장 `sleep()` 함수와 유사하지만, `Timer::sleep()`은 블로킹되지 않으며(현재 프로세스를 차단하지 않음)입니다.

> **주의**
> 이 기능은 workerman>=5.0이 필요합니다.
> 이 기능을 사용하려면 composer require revolt/event-loop ^1.0.0를 설치해야 하거나 이벤트 드라이버로 Swoole/Swow를 사용해야 합니다.


### 매개변수
``` delay ```

몇 초마다 실행할지를 나타내며, 소숫점을 지원하며 0.001초, 즉 밀리 초 단위까지 정확도를 가질 수 있습니다.

### 반환값
반환값 없음

### 예제

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    // 1.5초 딜레이 후 데이터 전송
    Timer::sleep(1.5);
    // 데이터 전송
    $connection->send('hello workerman');
};

Worker::runAll();
```
