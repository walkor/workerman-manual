# Composant serveur Channel

**```(Workerman >= 3.3.0)```**

# __construct 
```php
void \Channel\Server::__construct([string $listen_ip = '0.0.0.0', int $listen_port = 2206])
```

Instancie un serveur \Channel\Server

### Paramètres
``` listen_ip ```

Adresse IP locale à écouter, par défaut est ```0.0.0.0```

``` listen_port ```

Port à écouter, par défaut est 2206

## Exemple

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Si aucun paramètre n'est passé, la valeur par défaut est d'écouter sur 0.0.0.0:2206
$channel_server = new Channel\Server();

if(!defined('GLOBAL_START'))
{
    Worker::runAll();
}
```
