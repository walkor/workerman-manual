# utilisateur

## Description:
```php
string Worker::$user
```

Définit l'utilisateur sous lequel l'instance actuelle de Worker fonctionnera. Cette propriété n'a d'effet que si l'utilisateur actuel est root. Par défaut, l'instance fonctionne sous l'utilisateur actuel s'il n'est pas défini.

Il est recommandé de définir `$user` avec un utilisateur ayant des privilèges plus bas, par exemple www-data, apache, nobody, etc.

Remarque : cette propriété doit être définie avant l'exécution de `Worker::runAll();`. Cette fonctionnalité n'est pas prise en charge par le système Windows.

## Exemple

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Définir l'utilisateur de l'instance
$worker->user = 'www-data';
$worker->onWorkerStart = function($worker)
{
    echo "Démarrage du worker...\n";
};
// Exécuter le worker
Worker::runAll();
```
