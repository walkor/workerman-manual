# defaultMaxSendBufferSize
## Описание:
```php
static int Connection::$defaultMaxSendBufferSize
```

Это глобальный статический атрибут, используемый для установки размера буфера отправки по умолчанию для всех соединений. Если не установлен, по умолчанию установлено значение ```1MB```. Значение ```Connection::$defaultMaxSendBufferSize``` может быть динамически установлено, и установленное значение применяется только к новым соединениям, установленным после этого.

Это свойство влияет на обратный вызов [onBufferFull](../worker/on-buffer-full.md).

## Пример

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Устанавливаем размер буфера отправки по умолчанию для всех соединений
TcpConnection::$defaultMaxSendBufferSize = 2*1024*1024;

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // Устанавливаем размер буфера отправки приложения для текущего соединения, перекрывая значение по умолчанию
    $connection->maxSendBufferSize = 4*1024*1024;
};
// Запускаем worker
Worker::runAll();
```
