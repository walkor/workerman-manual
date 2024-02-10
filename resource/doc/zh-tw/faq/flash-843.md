# 開啟 Flash 的 843 端口

當 Flash 試圖與遠端服務器建立 socket 連接時，首先會向對應服務器的 843 端口請求一個安全策略文件。否則 Flash 將無法與服務器建立連接。在 webman 中，您可以使用以下方法開啟一個 843 端口，並返回安全策略文件。

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

在這裡，您可以根據需要自訂 XML 安全策略的內容。
