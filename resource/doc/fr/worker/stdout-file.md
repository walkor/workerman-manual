# stdoutFile
## Description:
```php
static string Worker::$stdoutFile
```

Cette propriété est une propriété statique globale. Si l'exécution est lancée en mode daemon (avec ```-d```), toutes les sorties vers le terminal (echo var_dump, etc.) seront redirigées vers le fichier spécifié par stdoutFile.

Si elle n'est pas définie et que l'exécution est en mode daemon, toutes les sorties du terminal seront redirigées vers ```/dev/null``` (c'est-à-dire que toutes les sorties seront par défaut supprimées).

> Remarque : ```/dev/null``` est un fichier spécial sous Linux, qui est en fait un trou noir. Toutes les données écrites dans ce fichier seront jetées. Si vous ne voulez pas supprimer les sorties, vous pouvez définir ```Worker::$stdoutFile``` comme un chemin de fichier normal.

> Remarque : Cette propriété doit être définie avant l'exécution de ```Worker::runAll();```. Cette fonctionnalité n'est pas prise en charge par le système Windows.

## Exemple

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$daemonize = true;
// Toutes les sorties sont enregistrées dans le fichier /tmp/stdout.log
Worker::$stdoutFile = '/tmp/stdout.log';
$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Démarrage du Worker\n";
};
// Exécution du worker
Worker::runAll();
```
