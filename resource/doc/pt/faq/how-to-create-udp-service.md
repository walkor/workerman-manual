# Como criar um serviço UDP

Em workerman, é muito simples criar um serviço UDP, como mostrado no código abaixo:

```php
$udp_worker = new Worker('udp://0.0.0.0:9292');
$udp_worker->onMessage = function($connection, $data){
    var_dump($data);
    $connection->send('get');
};
Worker::runAll();
```

Observação: Como o UDP é sem conexão, os serviços UDP não possuem eventos onConnect e onClose.
