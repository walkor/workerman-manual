# disiscriversi
**(Richiede Workerman versione >= 3.3.0)**

```php
void \Channel\Client::unsubscribe(string $event_name)
```
Annulla l'iscrizione a un determinato evento in modo che, quando si verifica l'evento, non verrà più attivato il callback ```$callback``` registrato tramite ```on($event_name, $callback)```.

### Parametri
``` $event_name ```

Nome dell'evento

### Valore restituito
void

### Esempio
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
