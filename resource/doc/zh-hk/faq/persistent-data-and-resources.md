# 物件和資源的永久化
在傳統的 Web 開發中，PHP 創建的物件、數據、資源等在請求結束後會全部釋放，造成難以實現永久化。然而在 Workerman 中可以輕鬆實現這些。

在 Workerman 中，如果想在記憶體中永久保存某些數據資源，可以將資源放到全域變量中或者類的靜態成員中。


例如下面的代碼：

使用一個全域變量```$connection_count```來保存當前進程的客戶端連接數。

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 全域變量，保存當前進程的客戶端連接數
$connection_count = 0;

$worker = new Worker('tcp://0.0.0.0:1236');

$worker->onConnect = function(TcpConnection $connection)
{
    // 有新的客戶端連接時，連接數+1
    global $connection_count;
    ++$connection_count;
    echo "now connection_count=$connection_count\n";
};

$worker->onClose = function(TcpConnection $connection)
{
    // 客戶端關閉時，連接數-1
    global $connection_count;
    $connection_count--;
    echo "now connection_count=$connection_count\n";
};

```


## PHP 變量作用域參見：
https://php.net/manual/zh/language.variables.scope.php
