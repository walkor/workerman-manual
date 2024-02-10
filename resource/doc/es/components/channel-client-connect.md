# Conectar
**``` (Requiere versión de Workerman >=3.3.0) ```**
```php
void \Channel\Client::connect([string $listen_ip = '127.0.0.1', int $listen_port = 2206])
```
Conecta al Channel/Server

### Parámetros
 ``` listen_ip ```

Dirección IP en la que el Channel/Server está escuchando, si no se proporciona, el valor por defecto es ```127.0.0.1```

 ``` listen_port ```

Puerto en el que el Channel/Server está escuchando, si no se proporciona, el valor por defecto es 2206

### Valor de retorno
void



### Ejemplo
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
