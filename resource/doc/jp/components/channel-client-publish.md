＃公開
**```（要求Workerman版本>=3.3.0）```**

```php
void \Channel\Client::publish(string $event_name, mixed $event_data)
```
特定のイベントを公開し、そのイベントを購読しているすべての購読者がこのイベントを受け取り、```on($event_name, $callback)```で登録されたコールバック```$callback```がトリガーされます。

### パラメータ
 ``` $event_name ```

公開するイベントの名前で、任意の文字列にすることができます。イベントに購読者がいない場合、イベントは無視されます。

 ``` $event_data ```

イベントに関連するデータで、数値、文字列、または配列にすることができます。

### 返り値
void

### 例
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
