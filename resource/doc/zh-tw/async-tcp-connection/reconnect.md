# reConnect 方法
```php
void AsyncTcpConnection::reConnect(float $delay = 0)
```
``` (要求Workerman版本>=3.3.5) ```
重新連線。通常在```onClose```回調中調用，實現斷線重連。

由於網絡問題或對方服務重啟等原因導致連接斷開，則可以通過調用此方法實現重連。

### 參數
``` $delay ```
延遲多久後執行重連。單位為秒，支持小數，可精確到毫秒。

如果不傳或者值為0則代表立即重連。

最好傳遞參數讓重連延遲執行，避免因為對端服務問題一直不可連導致本機CPU消耗過高。

### 返回值
無返回值

### 示例

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
        // 如果連接斷開，則在1秒後重連
        $con->reConnect(1);
    };
    $con->connect();
};

Worker::runAll();
```

> **注意**
> 調用reconnect成功重連後，$con的onConnect方法會再次被調用(如果有設置的話)。有時候我們想讓onConnect方法只執行一次，重連的時候不再執行，參考如下例子：

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
        // 如果連接斷開，則在1秒後重連
        $con->reConnect(1);
    };
    $con->connect();
};

Worker::runAll();
```
