# cancelar a inscrição
**``` (Requer Workerman versão >= 3.3.0) ```**

```php
void \Channel\Client::unsubscribe(string $event_name)
```
Cancela a inscrição em um determinado evento, de forma que quando esse evento ocorrer, não acionará mais o retorno de chamada ```$callback``` registrado em ```on($event_name, $callback)```.

### Parâmetros
``` $event_name ```

Nome do evento

### Valor de retorno
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
    $event_name = 'user_login';
    Channel\Client::on($event_name, function($event_data){
        var_dump($event_data);
    });
    Channel\Client::unsubscribe($event_name);
};

Worker::runAll();
```
