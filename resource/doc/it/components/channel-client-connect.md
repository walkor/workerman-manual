# connect
**``` (Richiede una versione di Workerman >=3.3.0) ```**
```php
void \Channel\Client::connect([string $listen_ip = '127.0.0.1', int $listen_port = 2206])
```
Connette al Server del Canale (Channel/Server).

### Parametri
``` listen_ip ```

L'indirizzo IP su cui il Server del Canale (Channel/Server) è in ascolto. Se non specificato, di default è ```127.0.0.1```.

``` listen_port ```

La porta su cui il Server del Canale (Channel/Server) è in ascolto. Se non specificato, di default è 2206.

### Valore restituito
void

### Esempio
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->onWorkerStart = function()
{
    Channel\Client::connect('127.0.0.1', 2206);
};

Worker::runAll();
```
