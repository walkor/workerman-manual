# Verbinden
**``` (Erfordert Workerman Version >= 3.3.0) ```**
```php
void \Channel\Client::connect([string $listen_ip = '127.0.0.1', int $listen_port = 2206])
```
Verbindung zum Channel/Server herstellen

### Parameter
 ``` listen_ip ```

Die IP-Adresse, die vom Channel/Server abgehört werden soll. Standardmäßig ist dies ```127.0.0.1```

 ``` listen_port ```

Der Port, der vom Channel/Server abgehört werden soll. Standardmäßig ist dies 2206

### Rückgabewert
void



### Beispiel
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
