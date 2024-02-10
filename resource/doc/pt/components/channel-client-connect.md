# conectar
**``` (requer versão Workerman >=3.3.0) ```**
```php
void \Channel\Client::connect([string $listen_ip = '127.0.0.1', int $listen_port = 2206])
```
Conecta ao Channel/Server

### Parâmetros
 ``` listen_ip ```
O endereço IP que o Channel/Server está ouvindo. O padrão é ```127.0.0.1```

 ``` listen_port ```
A porta que o Channel/Server está ouvindo. O padrão é 2206

### Valor retornado
void

### Exemplo
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
