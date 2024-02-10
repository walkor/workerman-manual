Flashにポート843を開く

Flashは、リモートサーバにソケット接続を要求する際に、まず対応するサーバの843ポートにセキュリティポリシーファイルを要求します。それ以外の場合、Flashはサーバとの接続を確立できません。Workermanでは、843ポートを開いてセキュリティポリシーファイルを返すために次のような方法を使用できます。

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

ここで、XMLのセキュリティポリシーの内容は、必要に応じてカスタマイズできます。
