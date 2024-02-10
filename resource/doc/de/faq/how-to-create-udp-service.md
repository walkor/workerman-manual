# Wie man einen UDP-Dienst einrichtet

Es ist sehr einfach, in Workerman einen UDP-Dienst einzurichten, ähnlich dem folgenden Code:

```php
$udp_worker = new Worker('udp://0.0.0.0:9292');
$udp_worker->onMessage = function($connection, $data){
    var_dump($data);
    $connection->send('get');
};
Worker::runAll();
```

Bitte beachten Sie: Da UDP verbindungslos ist, gibt es keine onConnect- und onClose-Ereignisse für UDP-Dienste.
