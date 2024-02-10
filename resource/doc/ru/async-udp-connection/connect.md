```php
void AsyncUdpConnection::connect()
```
Выполняет асинхронное соединение. Этот метод сразу же возвращает.

### Параметры
Нет параметров

### Возвращаемое значение
Нет возвращаемого значения

### Пример

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';


$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // Через 1 секунду запускаем udp-клиент, соединяемся с портом 1234 и отправляем строку "hi"
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // Получаем данные от сервера "hello"
            echo "recv $data\r\n";
            // Закрываем соединение
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // Получаем данные от клиента AsyncUdpConnection, отправляем строку "hello"
    $connection->send("hello");
};
Worker::runAll();             
```

После выполнения вывод будет примерно следующим образом:
```
recv hello
```
