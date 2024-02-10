# maxSendBufferSize
## Описание:
```php
int Connection::$maxSendBufferSize
```

У каждого соединения есть отдельный буфер отправки на уровне приложения. Если скорость приема клиента меньше скорости отправки сервера, данные будут временно храниться в буфере приложения в ожидании отправки.

Этот атрибут используется для установки размера буфера отправки на уровне приложения для текущего соединения. По умолчанию не установлено и равно [Connection::defaultMaxSendBufferSize](default-max-send-buffer-size.md) (1 МБ).

Этот атрибут влияет на обратный вызов [onBufferFull](../worker/on-buffer-full.md).

## Пример

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // Установить размер буфера отправки на уровне приложения для текущего соединения равным 102400 байт
    $connection->maxSendBufferSize = 102400;
};
// Запустить worker
Worker::runAll();
```
