# Cómo crear un servicio UDP

Crear un servicio UDP en workerman es muy sencillo, similar al siguiente código.

```php
$udp_worker = new Worker('udp://0.0.0.0:9292');
$udp_worker->onMessage = function($connection, $data){
    var_dump($data);
    $connection->send('get');
};
Worker::runAll();
```

Nota: Debido a que UDP es sin conexión, el servicio UDP no tiene eventos onConnect y onClose.
