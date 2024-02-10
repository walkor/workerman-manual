# Протокол ws

В настоящее время версия протокола **ws в Workerman составляет 13**.

Workerman может действовать в качестве клиента и инициировать подключение через протокол ws к удаленному веб-сокет серверу для двусторонней связи.

> **Обратите внимание**
> Протокол ws может быть использован только через AsyncTcpConnection в качестве клиента и не может слушать протокол веб-сокет сервера. То есть следующая запись является неверной.

```php
$worker = new Worker('ws://0.0.0.0:8080');
```

Если вы хотите использовать Workerman в качестве сервера веб-сокетов, используйте [протокол веб-сокетов](about-websocket.md).

**Пример использования протокола ws в качестве клиента веб-сокета:**

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Protocols\Ws;
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;

$worker = new Worker();
// При запуске процесса
$worker->onWorkerStart = function()
{
    // Подключение к удаленному веб-сокет серверу с помощью протокола веб-сокета
    $ws_connection = new AsyncTcpConnection("ws://127.0.0.1:1234");
    // Отправка сердечного импульса веб-сокет серверу с опкодом 0x9 каждые 55 секунд (необязательно)
    $ws_connection->websocketPingInterval = 55;
    // Установка заголовков HTTP (необязательно)
    $ws_connection->headers = [
        'Cookie' => 'PHPSID=82u98fjhakfusuanfnahfi; token=2hf9a929jhfihaf9i',
        'OtherKey' => 'values'
    ];
    // Установка типа данных (необязательно)
    $ws_connection->websocketType = Ws::BINARY_TYPE_BLOB; // BINARY_TYPE_BLOB для текста, BINARY_TYPE_ARRAYBUFFER для бинарных данных
    // После установки соединения TCP в результате трехстороннего рукопожатия (необязательно)
    $ws_connection->onConnect = function($connection){
        echo "tcp connected\n";
    };
    // После установки веб-сокет соединения (необязательно)
    $ws_connection->onWebSocketConnect = function(AsyncTcpConnection $con, $response) {
        echo $response;
        $con->send('hello');
    };
    // При получении сообщения от удаленного веб-сокет сервера
    $ws_connection->onMessage = function($connection, $data){
        echo "recv: $data\n";
    };
    // При возникновении ошибки соединения, обычно при неудачной попытке соединения с удаленным веб-сокет сервером (необязательно)
    $ws_connection->onError = function($connection, $code, $msg){
        echo "error: $msg\n";
    };
    // При отключении соединения с удаленным веб-сокет сервером (необязательно, рекомендуется выполнить повторное подключение)
    $ws_connection->onClose = function($connection){
        echo "connection closed and try to reconnect\n";
        // Если соединение разорвано, попытка повторного подключения через 1 секунду
        $connection->reConnect(1);
    };
    // После настройки всех вышеперечисленных обратных вызовов, выполняем операцию подключения
    $ws_connection->connect();
};
Worker::runAll();
```

Больше информации в разделе [Как использовать в качестве клиента ws/wss](../faq/as-wss-client.md).
