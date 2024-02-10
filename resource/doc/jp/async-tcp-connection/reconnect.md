```php
# reConnect メソッド
```php
void AsyncTcpConnection::reConnect(float $delay = 0)
```

``` (要求Workermanのバージョン>=3.3.5) ```

再接続。通常は```onClose```コールバックで呼び出され、切断した接続を再接続します。

ネットワークの問題や相手のサービス再起動などで接続が切断された場合は、このメソッドを呼び出すことで再接続できます。


### パラメーター
 ``` $delay ```

再接続を実行するまでの遅延時間。単位は秒で、小数をサポートし、ミリ秒まで精密に指定できます。

渡さないか、値が0の場合は即座に再接続を意味します。

再接続がCPU負荷が高すぎる際に、遅延して実行させるようなパラメーターを渡すことをお勧めします。


### 戻り値
戻り値はありません。

### 例

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
        // 接続が切断されたら、1秒後に再接続する
        $con->reConnect(1);
    };
    $con->connect();
};

Worker::runAll();
```

> **注意**
> reconnectを成功した場合、$conのonConnectメソッドが再度呼び出されます（設定されている場合）。再接続時にonConnectメソッドが1度だけ実行されるようにしたい場合は、次の例を参照してください。

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
        // 接続が切断されたら、1秒後に再接続する
        $con->reConnect(1);
    };
    $con->connect();
};

Worker::runAll();
```
