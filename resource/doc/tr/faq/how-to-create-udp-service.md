# UDP Hizmeti Nasıl Kurulur

Workerman'da UDP hizmeti kurmak oldukça basittir, aşağıdaki gibi bir kod yapısı kullanılır:

```php
$udp_worker = new Worker('udp://0.0.0.0:9292');
$udp_worker->onMessage = function($connection, $data){
    var_dump($data);
    $connection->send('get');
};
Worker::runAll();
```

Not: UDP bağlantısız olduğu için, UDP hizmetinde "onConnect" ve "onClose" etkinlikleri bulunmamaktadır.
