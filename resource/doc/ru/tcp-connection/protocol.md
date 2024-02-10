# Протокол

## Описание:
```php
string Connection::$protocol
```

Устанавливает протокол текущего соединения

## Пример

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    $connection->protocol = 'Workerman\\Protocols\\Http';
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    // при отправке данные автоматически вызывается $connection->protocol::encode(), упаковывает данные и отправляет их
    $connection->send("hello");
};
// Запуск worker
Worker::runAll();
```
