# Как клиент ws/wss

Иногда требуется использовать Workerman в качестве клиента для подключения к серверу по протоколу ws/wss и взаимодействия с ним.
Ниже приведены примеры.

## Workerman в качестве клиента ws

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){

    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    
    // После успешного выполнения рукопожатия websocket
    $con->onWebSocketConnect = function(AsyncTcpConnection $con, ) {
        $con->send('hello');
    };

    // При получении сообщения
    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## Workerman как клиент wss(ws+ssl)

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // ssl требует доступа к порту 443
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443');

    // Установка режима доступа с использованием ssl для создания wss
    $con->transport = 'ssl';

    $con->onWebSocketConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## Workerman в качестве клиента wss(ws+ssl) с локальным SSL-сертификатом

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // Настройка локального IP-адреса и порта, а также SSL-сертификата для доступа к удаленному хосту
    $context_option = array(
        // параметры ssl, см. http://php.net/manual/zh/context.ssl.php
        'ssl' => array(
            // Путь к локальному сертификату. Должен быть в формате PEM и содержать локальный сертификат и закрытый ключ.
            'local_cert'        => '/your/path/to/pemfile',
            // Пароль к файлу local_cert.
            'passphrase'        => 'your_pem_passphrase',
            // Разрешение на использование самоподписанных сертификатов.
            'allow_self_signed' => true,
            // Необходимость проверки SSL-сертификата.
            'verify_peer'       => false
        )
    );

    // ssl требует доступа к порту 443
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443', $context_option);

    // Установка режима доступа с использованием ssl для создания wss
    $con->transport = 'ssl';

    $con->onWebSocketConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## Другие настройки

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
    // Подключение к удаленному серверу по протоколу websocket
    $ws_connection = new AsyncTcpConnection("ws://127.0.0.1:1234");
    // Отправка сердечного импульса websocket с opcode 0x9 каждые 55 секунд
    $ws_connection->websocketPingInterval = 55;
    // Пользовательские заголовки HTTP
    $ws_connection->headers = ['token' => 'value'];
    // Установка типа данных по умолчанию, BINARY_TYPE_BLOB для текста
    $ws_connection->websocketType = Ws::BINARY_TYPE_BLOB; // BINARY_TYPE_BLOB для текста, BINARY_TYPE_ARRAYBUFFER для двоичных
    // После установки связи TCP
    $ws_connection->onConnect = function($connection){
        echo "tcp connected\n";
    };
    // После завершения рукопожатия websocket
    $ws_connection->onWebSocketConnect = function(AsyncTcpConnection $con, $response) {
        echo $response;
        $con->send('hello');
    };
    // При получении сообщения от удаленного сервера websocket
    $ws_connection->onMessage = function($connection, $data){
        echo "recv: $data\n";
    };
    // При возникновении ошибки, обычно при неудачном подключении к удаленному серверу websocket
    $ws_connection->onError = function($connection, $code, $msg){
        echo "error: $msg\n";
    };
    // При разрыве соединения с удаленным сервером websocket
    $ws_connection->onClose = function($connection){
        echo "connection closed and try to reconnect\n";
        // Если соединение разорвано, повторный подключение через 1 секунду
        $connection->reConnect(1);
    };
    // После установки всех вышеперечисленных обратных вызовов, выполнение подключения
    $ws_connection->connect();
};
Worker::runAll();
```

