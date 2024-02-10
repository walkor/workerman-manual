```php
void Connection::close(mixed $data = '')
```

Безопасно закрыть соединение и вызвать обратный вызов ```onClose``` для соединения.

Хотя UDP является безопасным для соединения, объект AsyncUdpConnection все еще хранится в памяти и должен быть закрыт с помощью метода close, иначе этот объект соединения по протоколу UDP будет постоянно существовать в памяти, что может привести к утечке памяти.

## Параметры

 ``` $data ```

Необязательный параметр, данные для отправки (если указан протокол, то данные ```$data``` будут автоматически упакованы с помощью метода encode протокола), после отправки данных соединение закроется, затем будет вызван обратный вызов onClose.

Размер данных не должен превышать 65507 байт, иначе отправка будет неудачной.

### Пример

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // Через 1 секунду создать UDP-клиента, подключиться к порту 1234 и отправить строку "hi"
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // Получить обратно данные "hello" от сервера
            echo "recv $data\r\n";
            // Закрыть соединение
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // Получить данные от клиента AsyncUdpConnection и вернуть строку "hello"
    $connection->send("hello");
};
Worker::runAll();             
```

Пример выполнения:

```plaintext
recv hello
```
