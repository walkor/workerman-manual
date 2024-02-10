# id
요구사항 ```(workerman >= 3.2.1)```


## 설명:
```php
int Worker::$id
```

현재 worker 프로세스의 ID 번호는  ```0```에서 ```$worker->count-1``` 까지의 범위에 있습니다.

이 속성은 worker 프로세스를 구별하는 데 매우 유용합니다. 예를 들어, 하나의 worker 인스턴스에 여러 프로세스가 있지만, 개발자는 한 프로세스에만 타이머를 설정하고 싶다면 ID를 식별하여 이를 수행할 수 있습니다. 예를 들어, worker 인스턴스의 ID 번호가 0인 프로세스에만 타이머를 설정하고 싶다면 (아래 예제 참조).

## 참고:

프로세스 다시 시작 후에도 ID 번호 값은 변경되지 않습니다.

프로세스 ID의 할당은 각 worker 인스턴스를 기반으로 합니다. 각 worker 인스턴스는 0부터 자체 프로세스에 ID 번호를 부여하므로 worker 인스턴스 간에는 ID 번호가 중복될 수 있지만 worker 인스턴스 내의 프로세스 ID 번호는 중복되지 않습니다. 다음 예제를 참조하십시오:

```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// worker 인스턴스 1에는 4개의 프로세스가 있으며, 프로세스 ID 번호는 각각 0, 1, 2, 3일 것입니다
$worker1 = new Worker('tcp://0.0.0.0:8585');
// 4개의 프로세스를 시작하도록 설정
$worker1->count = 4;
// 각 프로세스가 시작된 후 현재 프로세스 ID 번호 즉, $worker1->id를 출력합니다.
$worker1->onWorkerStart = function($worker1)
{
    echo "worker1->id={$worker1->id}\n";
};

// worker 인스턴스 2에는 2개의 프로세스가 있으며, 프로세스 ID 번호는 각각 0, 1일 것입니다
$worker2 = new Worker('tcp://0.0.0.0:8686');
// 2개의 프로세스를 시작하도록 설정
$worker2->count = 2;
// 각 프로세스가 시작된 후 현재 프로세스 ID 번호 즉, $worker2->id를 출력합니다.
$worker2->onWorkerStart = function($worker2)
{
    echo "worker2->id={$worker2->id}\n";
};

// worker 실행
Worker::runAll();
```

위와 같은 예시를 출력한 결과는 다음과 같습니다.
```
worker1->id=0
worker1->id=1
worker1->id=2
worker1->id=3
worker2->id=0
worker2->id=1
```

참고: Windows 시스템은 프로세스 수(count) 설정을 지원하지 않으므로, ID는 0번만을 가집니다. Windows 시스템에서는 동일한 파일로 두 개의 Worker를 초기화할 수 없으므로 이 예시는 Windows 시스템에서는 작동하지 않습니다.


## 예제
한 worker 인스턴스에는 4개의 프로세스가 있으며, ID 번호가 0인 프로세스에만 타이머를 설정합니다.

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8585');
$worker->count = 4;
$worker->onWorkerStart = function($worker)
{
    // ID 번호가 0인 프로세스에만 타이머를 설정하며, 다른 1, 2, 3번은 타이머를 설정하지 않습니다.
    if($worker->id === 0)
    {
        Timer::add(1, function(){
            echo "4개의 worker 프로세스 중 0번 프로세스에만 타이머 설정\n";
        });
    }
};
// worker 실행
Worker::runAll();
```
