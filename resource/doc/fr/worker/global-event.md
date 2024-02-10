# globalEvent

## Description:
```php
static Event Worker::$globalEvent
```

Cette propriété est une propriété statique globale qui représente une instance globale de la boucle d'événements, et permet d'enregistrer des événements de lecture/écriture de descripteur de fichier ou des événements de signal.

## Exemple

```php
use Workerman\Worker;
use Workerman\Events\EventInterface;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function($worker)
{
    echo 'Pid is ' . posix_getpid() . "\n";
    // Lorsque le processus reçoit le signal SIGALRM, afficher quelques informations
    Worker::$globalEvent->add(SIGALRM, EventInterface::EV_SIGNAL, function()
    {
        echo "Recevoir le signal SIGALRM\n";
    });
};
// Exécutez le worker
Worker::runAll();
```

## Test
Lorsque Workerman démarre, il affiche l'ID de processus actuel (un numéro). Exécutez la commande suivante dans le terminal
```bash
kill -SIGALRM PID_processus
```
Le serveur affichera alors
```bash
Recevoir le signal SIGALRM
```
