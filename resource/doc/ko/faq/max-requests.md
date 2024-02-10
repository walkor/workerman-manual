워커맨이 특정 요청을 처리한 후 현재 프로세스를 다시 시작하도록 설정하는 방법
Workerman을 보다 간략하게 만들기 위해 이러한 설정을 직접 제공하지는 않지만, 몇 줄의 코드로 이 기능을 구현할 수 있습니다.
```php
$worker->onMessage = function($connection, $data) {
    static $request_count;
    // 비즈니스 로직은 생략합니다.
    if(++$request_count > 10000) {
        // 요청 수가 10000에 도달하면 현재 프로세스를 종료하고, 주 프로세스가 자동으로 새 프로세스를 시작합니다.
        Worker::stopAll();
    }
};
```
