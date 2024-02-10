# publicar
**``` (Requer Workerman versão >= 3.3.0) ```**

```php
void \Channel\Client::publish(string $event_name, mixed $event_data)
```
Publica um evento específico, e todos os inscritos nesse evento receberão o evento e acionarão o callback ```$callback``` registrado por meio de ```on($event_name, $callback)```.

### Parâmetros
``` $event_name ```

Nome do evento a ser publicado, pode ser qualquer string. Se o evento não tem nenhum inscrito, o evento será ignorado.

``` $event_data ```

Dados relacionados ao evento, podem ser números, strings ou arrays.

### Valor de retorno
void

### Exemplo
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
    $event_name = 'user_login';
    $event_data = array('uid'=>123, 'uname'=>'tom');
    Channel\Client::publish($event_name, $event_data );
};

Worker::runAll();
```
