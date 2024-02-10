# publish
**``` (Workerman 버전 >= 3.3.0이 필요합니다) ```**

```php
void \Channel\Client::publish(string $event_name, mixed $event_data)
```
특정 이벤트를 발행하며, 해당 이벤트에 구독자가 있으면 해당 이벤트를 수신하고 ```on($event_name, $callback)```에 등록된 콜백 ```$callback```을 트리거합니다.

### 매개변수
 ``` $event_name ```

발행하는 이벤트의 이름으로, 임의의 문자열이 될 수 있습니다. 이벤트에 구독자가 없는 경우 이벤트는 무시될 것입니다.

 ``` $event_data ```

이벤트와 관련된 데이터로, 숫자, 문자열 또는 배열이 될 수 있습니다.

### 반환값
void

### 예제
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->onWorkerStart = function($http_worker)
{
    Channel\Client::connect('127.0.0.1', 2206);
};
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $event_name = 'user_login';
    $event_data = array('uid'=>123, 'uname'=>'tom');
    Channel\Client::publish($event_name, $event_data );
};

Worker::runAll();
```
