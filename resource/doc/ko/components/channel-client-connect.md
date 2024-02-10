# 연결
**``` (Workerman 버전 >= 3.3.0이 필요합니다) ```**
```php
void \Channel\Client::connect([string $listen_ip = '127.0.0.1', int $listen_port = 2206])
```
Channel/Server에 연결합니다.

### 매개변수
``` listen_ip ```

Channel/Server가 수신 대기하는 IP 주소입니다. 전달하지 않으면 기본값은 ```127.0.0.1```입니다.

``` listen_port ```

Channel/Server가 수신 대기하는 포트입니다. 전달하지 않으면 기본값은 2206입니다.

### 반환 값
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
};

Worker::runAll();
```
