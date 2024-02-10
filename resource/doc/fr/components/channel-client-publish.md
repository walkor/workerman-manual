# publier
**``` (Exige Workerman version >=3.3.0) ```**

```php
void \Channel\Client::publish(string $event_name, mixed $event_data)
```
Publie un événement particulier, tous les abonnés à cet événement recevront celui-ci et déclencheront le rappel ```$callback``` enregistré avec ```on($event_name, $callback)```.

### Paramètres
 ``` $event_name ```

Nom de l'événement publié, qui peut être n'importe quelle chaîne de caractères. Si l'événement n'a aucun abonné, il sera ignoré.

 ``` $event_data ```

Données liées à l'événement, qui peuvent être des nombres, des chaînes de caractères ou des tableaux.

### Valeur de retour
void

### Exemple
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
