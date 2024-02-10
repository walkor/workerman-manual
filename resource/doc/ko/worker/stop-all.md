# stopAll
```php
void Worker::stopAll(void)
```

현재 프로세스를 중지하고 종료합니다.

> **주의**
> `Worker::stopAll()`은 현재 프로세스를 중지하는 데 사용됩니다. 현재 프로세스가 종료되면 메인 프로세스가 즉시 새로운 프로세스를 시작합니다. 전체 workerman 서비스를 중지하려면 `posix_kill(posix_getppid(), SIGINT)`를 호출하십시오.

### 매개변수
매개변수 없음

### 반환 값
반환 값 없음

## 예제 max_request

아래 예제는 하위 프로세스가 1000개의 요청을 처리한 후 stopAll을 실행하여 종료하고 새로운 프로세스를 시작합니다. 이는 php-fpm의 max_request 속성과 유사하며 주로 php 비즈니스 코드 버그로 인한 메모리 누수 문제를 해결하는 데 사용됩니다.

start.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 각 프로세스에서 최대 1000개의 요청 처리
define('MAX_REQUEST', 1000);

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    // 처리된 요청 수
    static $request_count = 0;

    $connection->send('hello http');
    // 요청 수가 1000에 도달하면
    if(++$request_count >= MAX_REQUEST)
    {
        /*
         * 현재 프로세스를 종료하고 메인 프로세스가 곧바로 새로운 프로세스를 시작하여
         * 프로세스 재시작을 완료합니다.
         */
        Worker::stopAll();
    }
};

Worker::runAll();
```
