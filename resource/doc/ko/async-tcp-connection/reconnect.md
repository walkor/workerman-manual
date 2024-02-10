# reConnect 메소드

```php
void AsyncTcpConnection::reConnect(float $delay = 0)
```

``` (Workerman 버전 >= 3.3.5이상) ```

재연결합니다. 일반적으로 ```onClose``` 콜백에서 호출하여 연결이 끊어진 경우 재연결을 구현합니다.

네트워크 문제나 상대방 서비스 재시작으로 인해 연결이 끊긴 경우에는 이 메소드를 호출하여 재연결할 수 있습니다.

### 매개변수
``` $delay ```

재연결을 실행하기 전에 대기하는 시간. 초 단위의 지연으로, 소수점으로 정확도를 높일 수 있습니다.

전달하지 않거나 값이 0이면 즉시 재연결을 의미합니다.

재연결이 지연되어 실행되도록 매개변수를 전달하여 상대방 서비스 문제로 인해 계속하여 연결되어 있어서 CPU 소비가 과도하게 발생하는 것을 피할 수 있습니다.

### 반환 값
반환 값이 없습니다.

### 예시

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker)
{
    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };
    $con->onMessage = function(AsyncTcpConnection $con, $msg) {
        echo "recv $msg\n";
    };
    $con->onClose = function(AsyncTcpConnection $con) {
        // 연결이 끊어지면 1초 후 재연결
        $con->reConnect(1);
    };
    $con->connect();
};

Worker::runAll();
```

> **주의**
>  재연결 성공 후, $con의 onConnect 메소드가 다시 호출됩니다(설정된 경우). 때로는 재연결 시 onConnect 메소드를 한 번만 실행하고 다음에는 실행하지 않기를 원할 수 있습니다. 아래 예시를 참고하세요.

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker)
{
    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    $con->onConnect = function(AsyncTcpConnection $con) {
        static $is_first_connect = true;
        if (!$is_first_connect) return;
        $is_first_connect = false;
        $con->send('hello');
    };
    $con->onMessage = function(AsyncTcpConnection $con, $msg) {
        echo "recv $msg\n";
    };
    $con->onClose = function(AsyncTcpConnection $con) {
        // 연결이 끊어지면 1초 후 재연결
        $con->reConnect(1);
    };
    $con->connect();
};

Worker::runAll();
```
