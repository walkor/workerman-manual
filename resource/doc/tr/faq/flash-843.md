# Flash için 843 portunu etkinleştirme

Flash uzaktaki sunucuya soket bağlantısı başlatırken, önce ilgili sunucunun 843 portuna güvenlik politikası dosyası isteyecektir. Aksi takdirde Flash, sunucu ile bağlantı kuramaz. Workerman'da aşağıdaki gibi bir şekilde 843 portunu açıp güvenlik politikası dosyası döndürebilirsiniz.

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

XML güvenlik politikası içeriği isteğinize göre özelleştirilebilir.
