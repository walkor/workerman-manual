```php
# Свойство transport
```Requires (workerman >= 3.3.4)```

Установка свойства передачи, доступные значения [tcp](https://baike.baidu.com/subview/32754/8048820.htm) и [ssl](https://baike.baidu.com/view/525499.htm), по умолчанию tcp.

При transport [ssl](https://baike.baidu.com/view/525499.htm) PHP должен иметь установленное [openssl расширение](https://php.net/manual/zh/book.openssl.php).

Когда Workerman используется в качестве клиента для установки шифрованного ssl-соединения с сервером (https-соединение, wss-соединение и т.д.), установите эту опцию как `ssl`, как показано в примере ниже.

### Пример (https соединение)
```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// Асинхронное установление соединения с www.baidu.com при запуске процесса и отправка запроса на получение данных
$task->onWorkerStart = function($task)
{
    $connection_to_baidu = new AsyncTcpConnection('tcp://www.baidu.com:443');

    // Установка ssl-защищенного соединения
    $connection_to_baidu->transport = 'ssl';

    $connection_to_baidu->onConnect = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "Успешное подключение\n";
        $connection_to_baidu->send("GET / HTTP/1.1\r\nHost: www.baidu.com\r\nConnection: keep-alive\r\n\r\n");
    };
    $connection_to_baidu->onMessage = function(AsyncTcpConnection $connection_to_baidu, $http_buffer)
    {
        echo $http_buffer;
    };
    $connection_to_baidu->onClose = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "Соединение закрыто\n";
    };
    $connection_to_baidu->onError = function(AsyncTcpConnection $connection_to_baidu, $code, $msg)
    {
        echo "Ошибка код:$code сообщение:$msg\n";
    };
    $connection_to_baidu->connect();
};

// Запуск worker
Worker::runAll();
```
