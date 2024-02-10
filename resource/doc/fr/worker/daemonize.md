# Daemonize
## Description:
```php
static bool Worker::$daemonize
```

Cette propriété est une propriété statique globale indiquant si le programme est exécuté en tant que démon. Si la commande de démarrage utilise l'option ```-d```, alors cette propriété est automatiquement définie sur true. Il est également possible de la définir manuellement dans le code.

Remarque : cette propriété doit être définie avant l'exécution de ```Worker::runAll();```. Cette fonctionnalité n'est pas prise en charge par le système Windows.

## Exemple

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$daemonize = true;
$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Démarrage du travailleur\n";
};
// Exécuter le travailleur
Worker::runAll();
```
