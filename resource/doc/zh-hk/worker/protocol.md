# 協議

要求```（workerman >= 3.2.7）```

## 說明:
```php
string Worker::$protocol
```

設置當前Worker實例的協議類。

註：協議處理類可以直接在初始化Worker在監聽參數時直接指定。例如
```php
$worker = new Worker('http://0.0.0.0:8686');
```



## 範例


```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8686');
$worker->protocol = 'Workerman\\Protocols\\Http';

$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    $connection->send("hello");
};

// 運行worker
Worker::runAll();
```

以上程式碼等價於下面程式碼


```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

/**
 * 首先會查看用戶是否有自定義\Protocols\Http協議類，
 * 如果沒有使用workerman內置協議類Workerman\Protocols\Http
 */
$worker = new Worker('http://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    $connection->send("hello");
};

// 運行worker
Worker::runAll();
```
