# daemonize
## Descrizione
```php
static bool Worker::$daemonize
```

Questa è una proprietà statica globale che indica se il processo deve essere eseguito come un daemon. Se il comando di avvio utilizza l'opzione ```-d```, allora questa proprietà verrà automaticamente impostata su true. In alternativa, può essere impostata manualmente nel codice.

Nota: Questa proprietà deve essere impostata prima di eseguire ```Worker::runAll();```. Questa funzionalità non è supportata su sistemi Windows.

## Esempio

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$daemonize = true;
$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Avvio del worker\n";
};
// Esecuzione del worker
Worker::runAll();
```
