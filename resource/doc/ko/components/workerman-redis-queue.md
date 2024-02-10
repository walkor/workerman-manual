# workerman/redis-queue

Redis를 기반으로 한 메시지 대기열, 메시지 지연 처리를 지원합니다.

## 프로젝트 주소:
https://github.com/walkor/redis-queue

## 설치:
```composer require workerman/redis-queue
```

## 예제
```php
<?php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\RedisQueue\Client;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function () {
    $client = new Client('redis://127.0.0.1:6379');
   // 구독
    $client->subscribe('user-1', function($data){
        echo "user-1\n";
        var_export($data);
    });
   // 구독
    $client->subscribe('user-2', function($data){
        echo "user-2\n";
        var_export($data);
    });
    // 대기열에 정기적으로 메시지 보내기
    Timer::add(1, function()use($client){
        $client->send('user-1', ['some', 'data']);
    });
};

Worker::runAll();
```

## API
  * <a href="#construct"><code>Client::<b>__construct()</b></code></a>
  * <a href="#send"><code>Client::<b>send()</b></code></a>
  * <a href="#subscribe"><code>Client::<b>subscribe()</b></code></a>
  * <a href="#unsubscribe"><code>Client::<b>unsubscribe()</b></code></a>

-------------------------------------------------------

<a name="construct"></a>
### __construct (string $address, [array $options])

인스턴스 생성

  * `$address`  `redis://ip:6379`와 같은 형식, 반드시 'redis'로 시작해야 함.

  * `$options`  아래 옵션을 포함:
    * `auth`: 인증 정보, 기본값 ''
    * `db`: db, 기본값 0
    * `max_attempts`: 실패 후 재시도 횟수, 기본값 5
    * `retry_seconds`: 재시도 시간 간격(초), 기본값 5

> 실패는 비즈니스가 예외 'Exception' 또는 'Error'를 throw하는 것을 의미합니다. 실패한 후 메시지는 재시도를 위해 지연 대기열에 놓이며, 재시도 횟수는 'max_attempts'에 의해 제어되고, 재시도 간격은 'retry_seconds'와 'max_attempts'에 의해 공동으로 제어됩니다. 예를 들어, 'max_attempts'가 5이고, 'retry_seconds'가 10인 경우, 첫 번째 재시도 간격은 `1*10`초이고, 두 번째 재시도 간격은 `2*10`초이며, 세 번째 재시도 간격은 `3*10`초입니다.이와 같은 방식으로 5번까지 재시도합니다. 'max_attempts' 설정을 초과하면 메시지는`{redis-queue}-failed` (1.0.5 버전 이전에는 `redis-queue-failed`)라는 실패 대기열로 이동됩니다.

-------------------------------------------------------

<a name="send"></a>
### send(String $queue, Mixed $data, [int $dely=0])

대기열에 메시지 보내기

* `$queue` 대기열 이름, `String` 타입
* `$data` 게시된 구체적인 메시지, 배열 또는 문자열일 수 있음, `Mixed` 타입
* `$dely` 지연 소비 시간(초), 기본값 0, `Int` 타입

-------------------------------------------------------

<a name="subscribe"></a>
### subscribe(mixed $queue, callable $callback)

하나 또는 여러 대기열을 구독합니다.

* `$queue` 대기열 이름, 문자열 또는 여러 대기열 이름을 포함하는 배열일 수 있습니다.
* `$callback` 콜백 함수, 형식은 `function (Mixed $data)`로, 여기서 `$data`는 `send($queue, $data)`에서의 `$data`입니다.

-------------------------------------------------------

<a name="unsubscribe"></a>
### unsubscribe(mixed $queue)

구독을 취소합니다.

* `$queue` 대기열 이름 또는 여러 대기열 이름을 포함하는 배열일 수 있습니다.

-------------------------------------------------------

## 비-Workerman 환경에서 대기열에 메시지 보내기
일부 프로젝트가 아파치 또는 php-fpm 환경에서 실행 중이며 workerman/redis-queue 프로젝트를 사용할 수 없는 경우, 다음 함수를 참고하여 보낼 수 있습니다.
```php
function redis_queue_send($redis, $queue, $data, $delay = 0) {
    $queue_waiting = '{redis-queue}-waiting'; // 1.0.5 버전 이전에는 redis-queue-waiting
    $queue_delay = '{redis-queue}-delayed';// 1.0.5 버전 이전에는 redis-queue-delayed
    
    $now = time();
    $package_str = json_encode([
        'id'       => rand(),
        'time'     => $now,
        'delay'    => $delay,
        'attempts' => 0,
        'queue'    => $queue,
        'data'     => $data
    ]);
    if ($delay) {
        return $redis->zAdd($queue_delay, $now + $delay, $package_str);
    }
    return $redis->lPush($queue_waiting.$queue, $package_str);
}
```
여기서 `$redis`는 redis 인스턴스입니다. 예를 들어 redis 확장 사용 방법은 다음과 같습니다:
```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);
$queue = 'user-1';
$data= ['some', 'data'];
redis_queue_send($redis, $queue, $data);
```
