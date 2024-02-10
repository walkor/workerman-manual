# onMessage
## Описание:
```php
callback Worker::$onMessage
```

Функция обратного вызова, срабатывающая при получении данных от клиента через соединение (когда Workerman получает данные).

## Параметры функции обратного вызова

``` $connection ```

Объект соединения, то есть экземпляр [TcpConnection](../tcp-connection.md), используемый для управления клиентским соединением, такой как [отправка данных](../tcp-connection/send.md), [закрытие соединения](../tcp-connection/close.md) и т. д.

``` $data ```

Данные, присланные клиентским соединением. Если Worker определяет протокол, то $data является декодированными данными соответствующего протокола. Тип данных зависит от реализации метода `decode()` протокола, например, для `websocket`, `text` и `frame` данные представлены строкой, а для протокола HTTP - объектом [`Workerman\Protocols\Http\Request`](../http/request.md).

## Пример
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($data);
    $connection->send('receive success');
};
// Запуск worker
Worker::runAll();
```

Примечание: помимо использования анонимной функции в качестве обратного вызова, также можно [посмотреть здесь](../faq/callback_methods.md), где представлены другие способы обратного вызова.
