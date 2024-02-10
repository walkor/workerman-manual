# Componente server di Channel

**```(Richiede Workerman versione >=3.3.0)```**

# __construct
```php
void \Channel\Server::__construct([string $listen_ip = '0.0.0.0', int $listen_port = 2206])
```

Instanzia un server \Channel\Server

### Parametri
``` listen_ip ```

Indirizzo IP locale in ascolto, predefinito se non specificato è ```0.0.0.0```

``` listen_port ```

Porta in ascolto, predefinita se non specificata è 2206

## Esempio

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Se non specificati, verrà in ascolto su 0.0.0.0:2206 per impostazione predefinita
$channel_server = new Channel\Server();

if(!defined('GLOBAL_START'))
{
    Worker::runAll();
}
```
