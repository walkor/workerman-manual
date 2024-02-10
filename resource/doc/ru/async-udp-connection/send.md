# Метод send
```php
void AsyncUdpConnection::send(string $data)
```
Выполняет асинхронное соединение. Этот метод немедленно возвращает управление.

### Параметры
 ``` $data ```
Данные, отправляемые на сервер. Размер данных не может превышать 65507 байт (максимальный размер передачи одного UDP-пакета составляет 65507 байт), в противном случае отправка будет неудачной.

### Возвращаемые значения
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
    // Через 1 секунду запустить UDP-клиент, подключиться к порту 1234 и отправить строку hi
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // Получить данные, возвращенные сервером - hello
            echo "recv $data\r\n";
            // Закрыть соединение
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{   
    // Получить данные, отправленные клиентом AsyncUdpConnection, и вернуть строку hello
    $connection->send("hello");
};
Worker::runAll();              
```

После выполнения будет напечатано что-то вроде:
``` 
recv hello
```
