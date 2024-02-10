# del
```php
boolean \Workerman\Timer::del(int $timer_id)
```
Supprimer un minuteur spécifique

### Paramètres
``` timer_id ```

L'ID du minuteur, c'est-à-dire un entier retourné par l'interface add

### Valeur de retour
boolean

### Exemple
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// Activer plusieurs processus pour exécuter des tâches planifiées, attention aux problèmes de concurrence multi-processus
$task->count = 1;
$task->onWorkerStart = function(Worker $task)
{
    // Exécuter toutes les 2 secondes
    $timer_id = Timer::add(2, function()
    {
        echo "tâche exécutée\n";
    });
    // Exécuter une tâche unique après 20 secondes, supprimer la tâche planifiée toutes les 2 secondes
    Timer::add(20, function($timer_id)
    {
        Timer::del($timer_id);
    }, array($timer_id), false);
};

// Exécuter le worker
Worker::runAll();
```

### Exemple (Supprimer le minuteur actuel dans le rappel du minuteur)
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    // Remarque : il est nécessaire d'introduire l'ID actuel du minuteur dans le rappel en utilisant la méthode de référence (&)
    $timer_id = Timer::add(1, function()use(&$timer_id)
    {
        static $i = 0;
        echo $i++."\n";
        // Supprimer le minuteur après 10 exécutions
        if($i === 10)
        {
            Timer::del($timer_id);
        }
    });
};

// Exécuter le worker
Worker::runAll();
```
