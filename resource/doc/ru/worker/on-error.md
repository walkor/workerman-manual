# onError
## описание:
```php
callback Worker::$onError
```

Срабатывает, когда происходит ошибка соединения с клиентом.

В настоящее время типы ошибок включают:

1. Ошибка вызова Connection::send из-за разрыва соединения с клиентом (затем вызывается обратный вызов onClose) ```(code:WORKERMAN_SEND_FAIL msg:client closed)```

2. После срабатывания onBufferFull (буфер отправки заполнен), вызывается Connection::send, и буфер отправки остается полным, что приводит к сбою отправки (не вызовет обратного вызова onClose) ```(code:WORKERMAN_SEND_FAIL msg:send buffer full and drop package)```

3. Ошибка асинхронного соединения AsyncTcpConnection (затем вызывается обратный вызов onClose) ```(code:WORKERMAN_CONNECT_FAIL msg:stream_socket_client возвращает сообщение об ошибке)```

## Параметры функции обратного вызова
``` $connection ```

Объект соединения, то есть [экземпляр TcpConnection](../tcp-connection.md), используемый для управления клиентским соединением, такой как [отправка данных](../tcp-connection/send.md), [закрытие соединения](../tcp-connection/close.md) и т. Д.

``` $code ```

Код ошибки

``` $msg ```

Сообщение об ошибке

## пример
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onError = function(TcpConnection $connection, $code, $msg)
{
    echo "ошибка $code $msg\n";
};
// Запуск worker
Worker::runAll();
```

Примечание: помимо использования анонимной функции в качестве обратного вызова, также можно [посмотреть здесь](../faq/callback_methods.md) другие способы записи обратного вызова.
