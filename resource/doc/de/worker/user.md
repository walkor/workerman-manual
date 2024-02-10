# Benutzer

## Beschreibung:
```php
string Worker::$user
```

Legt fest, unter welchem Benutzer die aktuelle Worker-Instanz ausgeführt wird. Diese Eigenschaft ist nur dann wirksam, wenn der aktuelle Benutzer root ist. Standardmäßig läuft die Worker-Instanz als der aktuelle Benutzer.

Es wird empfohlen, dass `$user` mit einem Benutzer niedrigeren Rechten wie www-data, apache oder niemand festgelegt wird.

Hinweis: Diese Eigenschaft muss vor dem Aufruf von `Worker::runAll();` festgelegt werden, um wirksam zu sein. Dieses Feature wird nicht von Windows-Systemen unterstützt.

## Beispiel

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Legt den auszuführenden Benutzer für die Instanz fest
$worker->user = 'www-data';
$worker->onWorkerStart = function($worker)
{
    echo "Worker startet...\n";
};
// Führt den Worker aus
Worker::runAll();
```
