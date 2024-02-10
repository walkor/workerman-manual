# onWorkerStart
## 설명:
```php
callback Worker::$onWorkerStart
```

Worker 자식 프로세스가 시작될 때 실행되는 콜백 함수를 설정합니다. 각 자식 프로세스가 시작될 때마다 실행됩니다.

참고: onWorkerStart는 자식 프로세스가 시작될 때 실행되며, 여러 개의 자식 프로세스가 실행되는 경우(```$worker->count > 1```) 각 자식 프로세스마다 실행되므로 총 ```$worker->count```번 실행됩니다.


## 콜백 함수의 매개변수

``` $worker ```

Worker 객체


## 예시


```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStart = function(Worker $worker)
{
    echo "Worker starting...\n";
};
// worker 실행
Worker::runAll();
```

참고: 콜백으로 익명 함수를 사용하는 것 외에도 [여기](../faq/callback_methods.md)에서 다른 콜백 메소드를 사용하는 방법을 참고할 수 있습니다.
