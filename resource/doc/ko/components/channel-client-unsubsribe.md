# 구독 해지
**```(Workerman 버전>=3.3.0이 필요합니다)```**

```php
void \Channel\Client::unsubscribe(string $event_name)
```
특정 이벤트의 구독을 해지합니다. 이벤트가 발생할 때 더 이상 ```on($event_name, $callback)```에서 등록된 콜백 ```$callback```을 트리거하지 않습니다.

### 매개변수
 ``` $event_name ```

이벤트 이름

### 반환값
void

### 예시
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->onWorkerStart = function()
{
    Channel\Client::connect('127.0.0.1', 2206);
    $event_name = 'user_login';
    Channel\Client::on($event_name, function($event_data){
        var_dump($event_data);
    });
    Channel\Client::unsubscribe($event_name);
};

Worker::runAll();
```
