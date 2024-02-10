# onBufferDrain
## Описание:
```php
callback Worker::$onBufferDrain
```

У каждого соединения есть отдельный буфер передачи на уровне приложения, размер которого определяется переменной ```TcpConnection::$maxSendBufferSize```, значение по умолчанию составляет 1 МБ и может быть изменено вручную. Изменение размера применяется ко всем соединениям. 

Этот обратный вызов срабатывает после того, как все данные буфера передачи на уровне приложения были отправлены. Обычно используется совместно с onBufferFull, например, приостанавливать отправку данных на конечный узел при срабатывании onBufferFull, а затем возобновлять запись данных при срабатывании onBufferDrain.


## Параметры обратного вызова

 ``` $connection ```

Объект соединения, то есть [экземпляр TcpConnection](../tcp-connection.md), используется для управления клиентским соединением, таких как [отправка данных](../tcp-connection/send.md), [закрытие соединения](../tcp-connection/close.md) и т.д.


## Пример

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onBufferFull = function(TcpConnection $connection)
{
    echo "Буфер полон, дальнейшая отправка не выполняется\n";
};
$worker->onBufferDrain = function(TcpConnection $connection)
{
    echo "Буфер освободился, продолжение отправки\n";
};
// Запуск worker
Worker::runAll();
```

Примечание: Помимо использования анонимных функций в качестве обратного вызова, можно также [посмотреть здесь](../faq/callback_methods.md) другие способы написания обратных вызовов.

## См. также
onBufferFull - Срабатывает, когда буфер передачи на уровне приложения соединения полностью заполнен
