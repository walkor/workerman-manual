# Протокол WebSocket

В настоящее время версия **протокола WebSocket в Workerman составляет 13**.

Протокол WebSocket - это новый протокол HTML5. Он реализует двустороннюю связь между браузером и сервером.

## Взаимосвязь WebSocket и TCP

WebSocket, подобно HTTP, является протоколом прикладного уровня, основанным на передаче данных по TCP. Сам WebSocket не имеет прямого отношения к сокетам, тем более не может быть равнозначен.

## Рукопожатие по протоколу WebSocket

Протокол WebSocket предусматривает процесс рукопожатия, при котором браузер и сервер обмениваются данными по протоколу HTTP. В Workerman можно вмешаться в этот процесс следующим образом.

**Если версия Workerman <= 4.1**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Connection\TcpConnection;
use Workerman\Worker;

$ws = new Worker('websocket://0.0.0.0:8181');
$ws->onConnect = function($connection)
{
    $connection->onWebSocketConnect = function($connection , $httpBuffer)
    {
        // Здесь можно проверить, является ли источник подключения законным, и в случае неподходящего источника закрыть соединение
        // $_SERVER['HTTP_ORIGIN'] указывает на сайт, с которого было запрошено websocket-соединение
        if($_SERVER['HTTP_ORIGIN'] != 'https://www.workerman.net')
        {
            $connection->close();
        }
        // В onWebSocketConnect можно использовать $_GET и $_SERVER
        // var_dump($_GET, $_SERVER);
    };
};
Worker::runAll();
```

**Если версия Workerman >= 5.0**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Worker;

$worker = new Worker('websocket://0.0.0.0:12345');
$worker->onWebSocketConnect = function (TcpConnection $connection, Request $request) {
    if ($request->header('origin') != 'https://www.workerman.net') {
        $connection->close();
    }
    var_dump($request->get());
    var_dump($request->header());
};
Worker::runAll();
```

## Передача двоичных данных по протоколу WebSocket

По умолчанию протокол WebSocket может передавать только текст в кодировке UTF-8. Если необходимо передавать двоичные данные, обратитесь к следующему разделу.

В протоколе WebSocket используется маркировка в заголовке протокола для обозначения типа передаваемых данных: текстовые данные в UTF-8 или двоичные данные. Браузер проверяет маркер и соответствие типа данных, и, в случае несоответствия, может разорвать соединение.

Поэтому при отправке данных с сервера необходимо установить эту маркировку в зависимости от типа передаваемых данных. В Workerman, если это обычный текст в UTF-8, установите (обычно это устанавливается по умолчанию и не требует дополнительной настройки)
```php
use Workerman\Protocols\Websocket;
$connection->websocketType = Websocket::BINARY_TYPE_BLOB;
```

Если передаются двоичные данные, установите
```php
use Workerman\Protocols\Websocket;
$connection->websocketType = Websocket::BINARY_TYPE_ARRAYBUFFER;
```

**Обратите внимание**: если не установить $connection->websocketType, то по умолчанию $connection->websocketType будет равно BINARY_TYPE_BLOB (т.е. текст в UTF-8). Обычно приложения передают текстовые данные, например, данные в формате JSON, поэтому не нужно устанавливать $connection->websocketType. Это свойство необходимо устанавливать только при передаче двоичных данных (например, изображений, данных в формате Protobuf и т.д.).

## Использование Workerman в качестве клиента WebSocket

Можно использовать [класс AsyncTcpConnection](../async-tcp-connection.md) с протоколом [ws](about-ws.md), чтобы Workerman действовал в качестве клиента WebSocket, подключаясь к удаленному серверу WebSocket и обеспечивая двустороннюю реальном времени связь.
