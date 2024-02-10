# 物件和資源的持久化
在傳統的Web開發中，PHP所建立的物件、資料、資源等在請求完成後會全部釋放，這使得持久化變得非常困難。然而，在WorkerMan中，可以輕鬆實現這些功能。

在WorkerMan中，如果想要將某些資料資源永久保存在記憶體中，可以將資源放在全域變數或類別的靜態成員中。

舉例如下面的程式碼：

利用全域變數```$connection_count```來保存當前進程的客戶端連線數。

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 全域變數，保存當前進程的客戶端連線數
$connection_count = 0;

$worker = new Worker('tcp://0.0.0.0:1236');

$worker->onConnect = function(TcpConnection $connection)
{
    // 當有新的客戶端連線時，連線數+1
    global $connection_count;
    ++$connection_count;
    echo "now connection_count=$connection_count\n";
};

$worker->onClose = function(TcpConnection $connection)
{
    // 當客戶端關閉時，連線數-1
    global $connection_count;
    $connection_count--;
    echo "now connection_count=$connection_count\n";
};
```

## PHP變數作用域請參見：
https://php.net/manual/zh/language.variables.scope.php
