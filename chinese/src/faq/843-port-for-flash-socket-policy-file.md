# 为Flash开启843端口


Flash发起socket连接远程服务端时，首先会到对应服务端的843端口请求一个安全策略文件。否则Flash无法建立与服务端的连接。在Workerman中可以用如下方法开启一个843端口，返回安全策略文件。

```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/Workerman/Autoloader.php';

$flash_policy = new Worker('tcp://0.0.0.0:843');
$flash_policy->onMessage = function($connection, $message)
{
    $connection->send('<?xml version="1.0"?><cross-domain-policy><site-control permitted-cross-domain-policies="all"/><allow-access-from domain="*" to-ports="*"/></cross-domain-policy>'."\0");
};

if(!defined('GLOBAL_START'))
{
    Worker::runAll();
}
```

其中xml的安全策略内容可以根据你的需要进行自定义设置。
