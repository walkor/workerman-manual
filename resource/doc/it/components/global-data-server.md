# GlobalData Component Server
**``` (Richiede Workerman versione >=3.3.0) ```**

# __construct
```php
void \GlobalData\Server::__construct([string $listen_ip = '0.0.0.0', int $listen_port = 2207])
```

Istanziare un servizio \GlobalData\Server

### Parametri
 ``` listen_ip ```

Indirizzo IP locale in ascolto, predefinito è ```0.0.0.0```

 ``` listen_port ```

Porta in ascolto, predefinito è 2207


## Esempio
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Ascolta la porta
$worker = new GlobalData\Server('127.0.0.1', 2207);

Worker::runAll();
```
