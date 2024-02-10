# Componente del servidor de canal

**``` (Se requiere Workerman versión >= 3.3.0) ```**

# __construct
```php
void \Channel\Server::__construct([string $listen_ip = '0.0.0.0', int $listen_port = 2206])
```

Instancia un servidor \Channel\Server

### Parámetros
``` listen_ip ```

Dirección IP local a la que escuchar, si no se especifica, el valor predeterminado es ```0.0.0.0```

``` listen_port ```

Puerto al que escuchar, si no se especifica, el valor predeterminado es 2206

## Ejemplo

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Si no se especifican parámetros, el servidor escuchará en 0.0.0.0:2206
$channel_server = new Channel\Server();

if(!defined('GLOBAL_START'))
{
    Worker::runAll();
}
```
