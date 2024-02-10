# 啟用Flash的843端口

當Flash發起對遠端服務器的socket連接時，首先會向對應服務器的843端口請求一個安全策略文件。否則，Flash將無法與服務器建立連接。在Webman中，您可以使用以下方法啟用843端口，並返回安全策略文件。

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$flash_policy = new Worker('tcp://0.0.0.0:843');
$flash_policy->onMessage = function(TcpConnection $connection, $message)
{
    $connection->send('<?xml version="1.0"?><cross-domain-policy><site-control permitted-cross-domain-policies="all"/><allow-access-from domain="*" to-ports="*"/></cross-domain-policy>'."\0");
};

if(!defined('GLOBAL_START'))
{
    Worker::runAll();
}
```

其中XML安全策略內容可以根據您的需求進行自定義設置。
