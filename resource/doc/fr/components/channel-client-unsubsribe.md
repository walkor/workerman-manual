# Se désabonner
**``` (nécessite Workerman version>=3.3.0) ```**

```php
void \Channel\Client::unsubscribe(string $event_name)
```

Annule l'abonnement à un événement spécifique, de sorte que lorsque cet événement se produit, le rappel ```$callback``` enregistré par ```on($event_name, $callback)``` ne sera plus déclenché.

### Paramètres
 ``` $event_name ```

Nom de l'événement

### Valeur de retour
void



### Exemple
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
