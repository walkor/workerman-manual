# runAll
```php
void Worker::runAll(void)
```
모든 Worker 인스턴스를 실행합니다.

**주의사항:** 

Worker::runAll()을 실행하면 영원히 차단됩니다. 즉, Worker::runAll() 뒤에 있는 코드는 실행되지 않습니다. 모든 Worker 인스턴스화는 Worker::runAll() 이전에 이루어져야 합니다.

### 매개변수
매개변수 없음

### 반환값
반환값 없음

## 예제: 여러 개의 Worker 인스턴스 실행하기

start.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello http');
};

$ws_worker = new Worker('websocket://0.0.0.0:4567');
$ws_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello websocket');
};

// 모든 Worker 인스턴스 실행
Worker::runAll();
```


**주의사항:** 

Windows 버전의 Workerman은 동일한 파일에서 여러 개의 Worker를 인스턴스화하는 것을 지원하지 않습니다. 위의 예제는 Windows 버전의 Workerman에서 실행할 수 없습니다.

Windows 버전의 Workerman은 다음과 같이 각각의 파일에 다른 Worker 인스턴스를 초기화해야 합니다.

start_http.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello http');
};

// 모든 Worker 인스턴스 실행(여기서는 하나의 인스턴스만 있음)
Worker::runAll();
```

start_websocket.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$ws_worker = new Worker('websocket://0.0.0.0:4567');
$ws_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello websocket');
};

// 모든 Worker 인스턴스 실행(여기서는 하나의 인스턴스만 있음)
Worker::runAll();
```

