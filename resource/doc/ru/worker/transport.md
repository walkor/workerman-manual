# транспорт
## Описание:
```php
string Worker::$transport
```

Устанавливает используемый в настоящий момент экземпляром Worker протокол передачи данных. В настоящее время поддерживаются только три вида (tcp, udp, ssl). Если не установлено, по умолчанию используется tcp.

``` Заметка: ssl требуется версия Workerman >= 3.3.7 ```

## Пример 1

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:8484');
// Использование протокола udp
$worker->transport = 'udp';
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('Hello');
};
// Запуск worker-а
Worker::runAll();
```


## Пример 2

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Лучше использовать сертификат
$context = array(
    'ssl' => array(
        'local_cert' => '/etc/nginx/conf.d/ssl/server.pem', // Может также быть файлом crt
        'local_pk'   => '/etc/nginx/conf.d/ssl/server.key',
    )
);
// Здесь установлен протокол websocket
$worker = new Worker('websocket://0.0.0.0:4431', $context);
// Установка транспорта для включения ssl, websocket+ssl это wss
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```
