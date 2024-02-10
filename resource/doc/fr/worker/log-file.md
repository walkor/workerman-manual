# logFile
## Description:
```php
static string Worker::$logFile
```

Utilisé pour spécifier l'emplacement du fichier journal de Workerman.

Ce fichier enregistre les journaux liés à Workerman lui-même, tels que le démarrage, l'arrêt, etc.

Si aucun emplacement n'est spécifié, le nom du fichier par défaut est workerman.log et le fichier est situé dans le répertoire parent de Workerman.

**Remarque :**

Ce fichier journal ne contient que les journaux liés à Workerman lui-même, tels que le démarrage, l'arrêt, etc., et ne comprend aucun journal métier.

Les journaux métier peuvent être mis en œuvre à l'aide de fonctions telles que [file_put_contents](https://php.net/manual/zh/function.file-put-contents.php) ou [error_log](https://php.net/manual/zh/function.error-log.php).

## Example

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$logFile = '/tmp/workerman.log';

$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Démarrage du worker";
};
// Exécuter le worker
Worker::runAll();
```
