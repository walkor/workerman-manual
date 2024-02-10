```php
boolean \Workerman\Timer::del(int $timer_id)
```
일정 타이머 삭제

### 파라미터
 ``` timer_id ```

일정 타이머의 ID, 즉 add 인터페이스로 반환된 정수

### 반환 값
boolean


### 예시
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// 몇 개의 프로세스를 시작하여 일정 작업을 실행, 다중 프로세스 동시성 문제에 주의
$task->count = 1;
$task->onWorkerStart = function(Worker $task)
{
    // 매 2초마다 실행
    $timer_id = Timer::add(2, function()
    {
        echo "task run\n";
    });
    // 20초 후에 일회성 작업을 실행하고, 2초마다 실행되는 타이머를 삭제
    Timer::add(20, function($timer_id)
    {
        Timer::del($timer_id);
    }, array($timer_id), false);
};

// worker 실행
Worker::runAll();
```


### 예시(타이머 콜백에서 현재 타이머 삭제)
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    // 주의: 콜백 안에서 현재 타이머 ID를 사용할 때는 참조(&) 방식으로 가져와야 합니다.
    $timer_id = Timer::add(1, function()use(&$timer_id)
    {
        static $i = 0;
        echo $i++."\n";
        // 10번 실행 후 타이머 삭제
        if($i === 10)
        {
            Timer::del($timer_id);
        }
    });
};

// worker 실행
Worker::runAll();
```
