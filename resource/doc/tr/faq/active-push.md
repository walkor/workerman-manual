# Mesajları nasıl aktif olarak gönderebilirim

1. Zamanlayıcı kullanarak veri gönderme
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:1234');
// İşçi başlatıldıktan sonra istemciye düzenli aralıklarla veri gönder
$worker->onWorkerStart = function($worker){
    Timer::add(1, function()use($worker){
        foreach($worker->connections as $connection) {
            $connection->send('hello');
        }
    });
};
Worker::runAll();
```

2. Başka bir projede bir olay gerçekleştiğinde workerman'a veri gönderme
Daha fazla ayrıntı için [Sık Sorulan Sorular-Başka bir projede gönderme](push-in-other-project.md) sayfasına bakın.
