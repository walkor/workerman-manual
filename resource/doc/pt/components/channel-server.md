# Componente do Servidor do Canal

**``` (Requer Workerman versão >=3.3.0) ```**

# __construct
```php
void \Channel\Server::__construct([string $listen_ip = '0.0.0.0', int $listen_port = 2206])
```

Instancia um servidor \Channel\Server

### Parâmetros
``` listen_ip ```

Endereço IP local a ser ouvido, padrão é ```0.0.0.0```

``` listen_port ```

Porta a ser ouvida, padrão é 2206

## Exemplo

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Se nenhum parâmetro for passado, padrão é ouvir em 0.0.0.0:2206
$channel_server = new Channel\Server();

if(!defined('GLOBAL_START'))
{
    Worker::runAll();
}
```
