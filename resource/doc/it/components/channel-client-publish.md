# pubblicare
**``` (Richiede Workerman versione >=3.3.0) ```**

```php
void \Channel\Client::publish(string $nome_evento, mixed $dati_evento)
```
Pubblica un evento, tutti i sottoscrittori di questo evento riceveranno l'evento e attiveranno il callback ```$callback``` registrato con ```on($nome_evento, $callback)```.

### Parametri
 ``` $nome_evento ```

Il nome dell'evento pubblicato, può essere una stringa qualsiasi. Se non ci sono sottoscrittori per l'evento, verrà ignorato.

 ``` $dati_evento ```

Dati correlati all'evento, possono essere numeri, stringhe o array.

### Valore restituito
void



### Esempio
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
    $nome_evento = 'login_utente';
    $dati_evento = array('uid'=>123, 'uname'=>'tom');
    Channel\Client::publish($nome_evento, $dati_evento);
};

Worker::runAll();
```

