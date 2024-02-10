# How to establish UDP service

Setting up a UDP service in Workerman is quite simple, similar to the following code:

```php
$udp_worker = new Worker('udp://0.0.0.0:9292');
$udp_worker->onMessage = function($connection, $data){
    var_dump($data);
    $connection->send('get');
};
Worker::runAll();
```

Note: Because UDP is connectionless, UDP services do not have onConnect and onClose events.
