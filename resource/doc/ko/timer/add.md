```php
int \Workerman\Timer::add(float $time_interval, callable $callback [, $args = array() , bool $persistent = true])
```

특정 함수 또는 클래스 메소드를 주기적으로 실행합니다.

참고: 타이머는 현재 프로세스에서 실행되며 workerman은 새로운 프로세스나 스레드를 만들어 타이머를 실행하지 않습니다.

### 매개변수
``` time_interval ```

몇 초마다 한 번씩 실행할지를 나타냅니다. 초 단위이며 소수점을 지원하여 0.001단위로 정밀하게 지정할 수 있습니다.

``` callback ```

콜백 함수 ```참고: 콜백 함수가 클래스 메소드인 경우 메소드는 public 속성이어야 합니다.```

``` args ```

콜백 함수의 매개변수로, 배열 형태여야 하며 배열 요소는 매개변수 값입니다.

``` persistent ```

영구적인지 여부, 한 번만 실행하기를 원한다면 false를 전송합니다(한 번만 실행하는 작업은 실행이 완료되면 자동으로 파괴되며 ```Timer::del()```를 호출할 필요가 없습니다). 기본값은 true로 계속해서 주기적으로 실행합니다.

### 반환값
타이머의 timerid를 나타내는 정수를 반환합니다. 이를 통해 ```Timer::del($timerid)```를 호출하여 타이머를 파괴할 수 있습니다.

### 예시

#### 1. 익명 함수(클로저)로 타이머 설정
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->count = 1;
$task->onWorkerStart = function(Worker $task)
{
    $time_interval = 2.5;
    Timer::add($time_interval, function()
    {
        echo "작업 실행\n";
    });
};

Worker::runAll();
```
