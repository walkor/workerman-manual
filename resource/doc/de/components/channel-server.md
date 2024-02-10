# Channel Komponente Server

**``` (Workerman Version >=3.3.0 erforderlich) ```**

# __construct
```php
void \Channel\Server::__construct([string $listen_ip = '0.0.0.0', int $listen_port = 2206])
```

Instanziiert einen \Channel\Server Server.

### Parameter
``` listen_ip ```

Die IP-Adresse des lokalen Hosts, Standard ist ```0.0.0.0```

``` listen_port ```

Der zu lauschende Port, Standard ist 2206

## Beispiel

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Standardmäßig lauscht es auf 0.0.0.0:2206
$channel_server = new Channel\Server();

if(!defined('GLOBAL_START'))
{
    Worker::runAll();
}
```
