# Aktivieren des 843-Ports für Flash

Wenn Flash eine Socket-Verbindung zum Remote-Server herstellt, muss es zunächst eine Sicherheitsrichtliniendatei vom entsprechenden Server-Port 843 anfordern. Andernfalls kann Flash keine Verbindung zum Server herstellen. In Workerman kann ein Port 843 mit dem folgenden Code geöffnet werden, um die Sicherheitsrichtliniendatei zu senden.

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

Die Sicherheitsrichtlinien im XML-Format können je nach Bedarf angepasst werden.
