# nome

## Description:
```php
string Worker::$name
```

Imposta il nome dell'istanza corrente del Worker, per riconoscere il processo quando si esegue il comando di stato. Se non viene impostato, il nome predefinito è "none".

## Esempio

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Imposta il nome dell'istanza
$worker->name = 'MyWebsocketWorker';
$worker->onWorkerStart = function($worker)
{
    echo "Avvio del Worker...\n";
};
// Avvia il worker
Worker::runAll();
```
