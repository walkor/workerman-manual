# Comment établir un service UDP

Il est très simple d'établir un service UDP dans workerman, comme illustré dans le code suivant :

```php
$udp_worker = new Worker('udp://0.0.0.0:9292');
$udp_worker->onMessage = function($connection, $data){
    var_dump($data);
    $connection->send('get');
};
Worker::runAll();
```

Remarque : étant donné que l'UDP est sans connexion, le service UDP ne dispose pas des événements onConnect et onClose.
