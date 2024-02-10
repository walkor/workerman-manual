# Composant serveur GlobalData

**``` (Workerman version >=3.3.0 requise) ```**

# __construct
```php
void \GlobalData\Server::__construct([string $listen_ip = '0.0.0.0', int $listen_port = 2207])
```

Instancie un service \GlobalData\Server

### Paramètres
 ``` listen_ip ```

Adresse IP locale à écouter. Par défaut, c'est ```0.0.0.0```

 ``` listen_port ```

Port à écouter. Par défaut, c'est 2207

## Exemple
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Écouter le port
$worker = new GlobalData\Server('127.0.0.1', 2207);

Worker::runAll();
```
