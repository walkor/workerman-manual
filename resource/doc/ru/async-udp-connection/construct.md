# Метод __construct
```php
void AsyncUdpConnection::__construct(string $remote_address)
```
Создает объект соединения udp.

AsyncUdpConnection позволяет Workerman работать в качестве клиента для передачи udp данных с удаленным сервером.

## Параметры
Параметр: `remote_address`

Адрес соединения, например:
```
udp://192.168.1.1:1234
frame://192.168.1.1:8080
text://192.168.1.1:8080
```

## Пример

```php
use Workerman\Worker;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // Запустить udp клиент через 1 секунду, подключиться к порту 1234 и отправить строку "hi"
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function($udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function($udp_connection, $data){
            // Получить данные hello от сервера
            echo "recv $data\r\n";
            // Закрыть соединение
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function($connection, $data)
{
    // Получить данные от клиента AsyncUdpConnection и отправить строку "hello"
    $connection->send("hello");
};
Worker::runAll();             
```

После выполнения будет выведено что-то вроде:
```
recv hello
```
