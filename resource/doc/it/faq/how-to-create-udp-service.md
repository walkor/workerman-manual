# Come creare un servizio UDP

Creare un servizio UDP in workerman è molto semplice, simile al codice seguente:

```php
$udp_worker = new Worker('udp://0.0.0.0:9292');
$udp_worker->onMessage = function($connection, $data){
    var_dump($data);
    $connection->send('get');
};
Worker::runAll();
```

Nota: Poiché l'UDP è senza connessione, il servizio UDP non ha gli eventi onConnect e onClose.
