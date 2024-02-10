# Activer le port 843 pour Flash

Lorsque Flash tente de se connecter à un serveur distant via un socket, il commencera par demander un fichier de stratégie de sécurité sur le port 843 du serveur correspondant. Sans cela, Flash ne pourra pas établir de connexion avec le serveur. Dans Workerman, il est possible d'ouvrir un port 843 et de renvoyer le fichier de stratégie de sécurité de la manière suivante :

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

Le contenu de la politique de sécurité XML peut être personnalisé selon vos besoins.
