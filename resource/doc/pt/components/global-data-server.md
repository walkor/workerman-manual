# Componete do Servidor GlobalData
**``` (Requer Workerman versão >= 3.3.0) ```**

# __construct
```php
void \GlobalData\Server::__construct([string $listen_ip = '0.0.0.0', int $listen_port = 2207])
```

Instancia um serviço \GlobalData\Server

### Parâmetros
``` listen_ip ```

Endereço IP local para escutar, por padrão é ```0.0.0.0```

``` listen_port ```

Porta de escuta, por padrão é 2207


## Exemplo
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Escutar a porta
$worker = new GlobalData\Server('127.0.0.1', 2207);

Worker::runAll();
```
