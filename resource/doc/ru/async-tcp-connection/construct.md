```php
/**
 *  Метод __construct (конструктор)
 */

void AsyncTcpConnection::__construct(string $remote_address, $context_option = null)
```
Создает объект асинхронного подключения.

AsyncTcpConnection позволяет Workerman служить в качестве клиента для асинхронного подключения к удаленному серверу и отправлять и обрабатывать данные асинхронно через интерфейс send и обратного вызова onMessage.

## Параметры
Параметр: ``` remote_address ```

Адрес соединения, например
  ``` tcp://www.baidu.com:80 ```
  ``` ssl://www.baidu.com:443 ```
  ``` ws://echo.websocket.org:80 ```
  ``` frame://192.168.1.1:8080 ```
  ``` text://192.168.1.1:8080 ```

Параметр: ``` $context_option ```

  ```Этот параметр требуется (workerman >= 3.3.5)```

  Используется для установки контекста сокета, например, установка параметра ```bindto```, чтобы указать IP-адрес и порт (сетевой карты), доступ к внешней сети, установка SSL-сертификата и другие.

  См. [stream_context_create](https://php.net/manual/en/function.stream-context-create.php), [Параметры контекста сокета](https://php.net/manual/zh/context.socket.php), [Параметры контекста SSL](https://php.net/manual/zh/context.ssl.php).

## Примечание

В настоящее время AsyncTcpConnection поддерживает протоколы [tcp](https://baike.baidu.com/subview/32754/8048820.htm), [ssl](https://baike.baidu.com/view/525499.htm), [ws](appendices/about-ws.md), [frame](appendices/about-frame.md), [text](appendices/about-text.md).

Также поддерживается пользовательский протокол, см. [Как создать пользовательский протокол](../protocols/how-protocols.md).

Протокол [ssl](https://baike.baidu.com/view/525499.htm) требует Workerman >=3.3.4 и установленное [расширение openssl](https://php.net/manual/zh/book.openssl.php).

В настоящее время не поддерживается использование AsyncTcpConnection для протокола [http](https://baike.baidu.com/view/9472.htm).

Можно использовать ```new AsyncTcpConnection('ws://...')``` для создания подключения к удаленному серверу через WebSocket, аналогично браузеру в рамках Workerman, см. [Пример](../appendices/about-ws.md). Однако нельзя использовать форму ```new AsyncTcpConnection('websocket://...')``` для подключения через протокол WebSocket.

## Примеры

### Пример 1: Асинхронный доступ к внешнему http-сервису
```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// Асинхронное установление соединения с www.baidu.com при запуске процесса и отправка данных для получения ответа
$task->onWorkerStart = function($task)
{
    // Прямая передача http-запроса неподдерживается, но можно использовать tcp для имитации передачи данных по протоколу http
    $connection_to_baidu = new AsyncTcpConnection('tcp://www.baidu.com:80');
    // При успешном установлении соединения отправляем данные http-запроса
    $connection_to_baidu->onConnect = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "connect success\n";
        $connection_to_baidu->send("GET / HTTP/1.1\r\nHost: www.baidu.com\r\nConnection: keep-alive\r\n\r\n");
    };
    $connection_to_baidu->onMessage = function(AsyncTcpConnection $connection_to_baidu, $http_buffer)
    {
        echo $http_buffer;
    };
    $connection_to_baidu->onClose = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "connection closed\n";
    };
    $connection_to_baidu->onError = function(AsyncTcpConnection $connection_to_baidu, $code, $msg)
    {
        echo "Error code:$code msg:$msg\n";
    };
    $connection_to_baidu->connect();
};

// Запуск воркера
Worker::runAll();
```

### Пример 2: Асинхронное подключение к внешнему WebSocket-сервису

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // Установка локального IP-адреса и порта для доступа к удаленному хосту (каждое сокет-соединение будет использовать локальный порт)
    $context_option = array(
        'socket' => array(
            // IP должен быть IP-адресом сетевого адаптера этого узла и должен иметь доступ к удаленному хосту, иначе будет недействителен
            'bindto' => '114.215.84.87:2333',
        ),
    );

    $con = new AsyncTcpConnection('ws://echo.websocket.org:80', $context_option);

    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

### Пример 3: Асинхронное подключение к внешнему wss-порту и установка локального SSL-сертификата

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // Установка локального IP-адреса и порта для доступа к удаленному хосту и установка параметров SSL
    $context_option = array(
        'socket' => array(
            // IP должен быть IP-адресом сетевого адаптера этого узла и должен иметь доступ к удаленному хосту, иначе будет недействителен
            'bindto' => '114.215.84.87:2333',
        ),
        // параметры SSL, см. https://php.net/manual/zh/context.ssl.php
        'ssl' => array(
            // путь к локальному сертификату. Должен быть в формате PEM и содержать локальные сертификаты и закрытый ключ.
            'local_cert'        => '/your/path/to/pemfile',
            // пароль для файла local_cert.
            'passphrase'        => 'your_pem_passphrase',
            // разрешение использовать самоподписанный сертификат.
            'allow_self_signed' => true,
            // необходимость проверки SSL-сертификата.
            'verify_peer'       => false
        )
    );

    // Инициировать асинхронное подключение
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443', $context_option);

    // Установить способ доступа через SSL
    $con->transport = 'ssl';

    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```
