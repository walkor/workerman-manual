# destroy
## Описание:
```php
void Connection::destroy()
```

Немедленно закрывает соединение.

В отличие от close, вызов destroy приведет к немедленному закрытию соединения, даже если в очереди отправки данного соединения еще остались данные, которые не были отправлены получателю. Также немедленно будет вызван обратный вызов ```onClose``` для данного соединения.

## Параметры

Без параметров


## Пример

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // при возникновении проблем
    $connection->destroy();
};
// Запуск worker
Worker::runAll();
```
