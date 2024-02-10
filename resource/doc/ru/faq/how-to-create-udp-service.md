# Как создать службу UDP

Создание службы UDP в workerman очень просто, подобно следующему коду:

```php
$udp_worker = new Worker('udp://0.0.0.0:9292');
$udp_worker->onMessage = function($connection, $data){
    var_dump($data);
    $connection->send('get');
};
Worker::runAll();
```

Обратите внимание: поскольку UDP не имеет соединения, служба UDP не имеет событий onConnect и onClose.
