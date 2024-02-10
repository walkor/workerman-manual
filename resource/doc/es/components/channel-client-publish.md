# publicar
**``` (Require Workerman versión >=3.3.0) ```**

```php
void \Channel\Client::publish(string $nombre_evento, mixed $datos_evento)
```
Publica un evento específico, todos los suscriptores de este evento recibirán el evento y activarán el callback ```$callback``` registrado con ```on($nombre_evento, $callback)```.

### Parámetros
``` $nombre_evento ```

El nombre del evento a publicar, puede ser cualquier cadena de texto. Si no hay suscriptores para el evento, será ignorado.

``` $datos_evento ```

Datos relacionados con el evento, pueden ser números, cadenas de texto o arreglos.

### Valor de retorno
void

### Ejemplo
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->onWorkerStart = function($http_worker)
{
    Channel\Client::connect('127.0.0.1', 2206);
};
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $nombre_evento = 'user_login';
    $datos_evento = array('uid'=>123, 'uname'=>'tom');
    Channel\Client::publish($nombre_evento, $datos_evento);
};

Worker::runAll();
```
