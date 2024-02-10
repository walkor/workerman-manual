# Enable Port 843 for Flash

When Flash initiates a socket connection to a remote server, it first requests a security policy file from the corresponding server's port 843. Otherwise, Flash cannot establish a connection to the server. In Workerman, you can enable port 843 and return a security policy file using the following method:

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

You can customize the content of the XML security policy according to your needs.
