# Как активно отправлять сообщения

1. Можно использовать таймер для плановой отправки данных
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:1234');
// После запуска процесса периодически отправлять данные клиентам
$worker->onWorkerStart = function($worker){
    Timer::add(1, function()use($worker){
        foreach($worker->connections as $connection) {
            $connection->send('hello');
        }
    });
};
Worker::runAll();
```

2. Уведомление workerman для отправки данных при возникновении события в другом проекте
См. [Часто задаваемые вопросы - Отправка в другом проекте](push-in-other-project.md)
