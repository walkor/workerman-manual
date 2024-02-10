# Componente de servidor GlobalData
**``` (Se requiere Workerman versión >= 3.3.0) ```**

# __construct
```php
void \GlobalData\Server::__construct([string $listen_ip = '0.0.0.0', int $listen_port = 2207])
```

Inicializa un servidor \GlobalData\Server

### Parámetros
 ``` listen_ip ```

Dirección IP local para escuchar, por defecto es ```0.0.0.0```

 ``` listen_port ```

Puerto de escucha, por defecto es 2207


## Ejemplo
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Escuchando el puerto
$worker = new GlobalData\Server('127.0.0.1', 2207);

Worker::runAll();
```
