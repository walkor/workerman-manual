# Abilitare la porta 843 per Flash

Quando Flash tenta di connettersi a un server remoto tramite socket, prima richiede al server corrispondente un file di politica di sicurezza sulla porta 843. In caso contrario, Flash non può stabilire una connessione con il server. In Workerman, è possibile aprire una porta 843 e restituire il file di politica di sicurezza come segue.

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

È possibile personalizzare il contenuto della politica di sicurezza XML in base alle proprie esigenze.
