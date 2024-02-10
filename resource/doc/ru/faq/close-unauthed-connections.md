# Закрытие неподтвержденного соединения
**Вопрос:**

Как закрыть соединение с клиентом, который не отправлял данные в течение определенного времени, например, закрыть соединение с клиентом, если в течение 30 секунд не было получено никаких данных, чтобы заставить неподтвержденные соединения подтвердиться в течение определенного времени?

**Ответ:**

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('xxx://x.x.x.x:x');
$worker->onConnect = function(TcpConnection $connection)
{
    // Временно добавляем свойство $auth_timer_id объекта $connection для хранения идентификатора таймера
    // Закрыть соединение через 30 секунд, если клиент не отправит подтверждение защиты за это время
    $connection->auth_timer_id = Timer::add(30, function()use($connection){
        $connection->close();
    }, null, false);
};
$worker->onMessage = function(TcpConnection $connection, $msg)
{
    $msg = json_decode($msg, true);
    switch($msg['type'])
    {
    case 'login':
        ...опущено
        // Успешное подтверждение, удаляем таймер, чтобы избежать закрытия соединения
        Timer::del($connection->auth_timer_id);
        break;
         ...опущено
    }
    ...опущено
}
```
