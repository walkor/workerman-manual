# Débogage de base

Workerman a deux modes de fonctionnement, le mode de débogage et le mode de fonctionnement en tant que démon.

Exécutez `php start.php start` pour entrer en mode débogage, dans ce cas, les fonctions d'impression telles que `echo, var_dump, var_export` dans le code seront affichées dans le terminal. Notez que lors de l'exécution de Workerman avec `php start.php start`, tous les processus se termineront lorsque le terminal sera fermé.

En revanche, l'exécution de `php start.php start -d` entre en mode démon, c'est-à-dire le mode de fonctionnement en ligne officielle, et la fermeture du terminal n'a aucun effet.

Si vous souhaitez voir les impressions des fonctions telles que `echo, var_dump, var_export` même en mode démon, vous pouvez définir la propriété Worker::$stdoutFile, par exemple

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Rediriger les sorties d'écran vers le fichier spécifié par Worker::$stdoutFile
Worker::$stdoutFile = '/tmp/stdout.log';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello world');
};

Worker::runAll();
```

De cette manière, toutes les impressions des fonctions telles que `echo, var_dump, var_export` seront écrites dans le fichier spécifié par `Worker::$stdoutFile`. Notez que le chemin spécifié par `Worker::$stdoutFile` doit avoir des autorisations d'écriture.
