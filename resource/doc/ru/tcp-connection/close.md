# close
## Описание:
```php
void Connection::close(mixed $data = '')
```

Безопасное закрытие соединения.

Вызов close будет ожидать завершения отправки данных из буфера перед закрытием соединения и вызовет обратный вызов ```onClose```.

## Параметры

 ``` $data ```

Необязательный параметр, данные для отправки (если указан протокол, автоматически вызывается метод кодирования протокола для упаковки данных ```$data```), после завершения отправки данных закрывается соединение, а затем вызывается обратный вызов onClose.


## Пример

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->close("hello\n");
};
// Запуск worker
Worker::runAll();
```
