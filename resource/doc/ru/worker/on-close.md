# onClose
## Описание:
```php
callback Worker::$onClose
```

Обратный вызов, который срабатывает, когда клиентское соединение разрывается с Workerman. Независимо от причины разрыва соединения, ```onClose``` будет вызван при любом разрыве соединения. Каждое соединение вызовет ```onClose``` только один раз.

Примечание: если соединение разорвано из-за экстремальных ситуаций, таких как потеря сети или отключение питания на стороне клиента, в этом случае, так как Workerman не может своевременно отправить tcp пакет о завершении соединения, Workerman не узнает, что соединение было разорвано, и не вызовет ```onClose``` вовремя. Эта ситуация требует реализации прикладного уровня мониторинга связи. Использование мониторинга соединения в Workerman описано [здесь](../faq/heartbeat.md). Если используется фреймворк GatewayWorker, достаточно использовать механизм мониторинга соединения в GatewayWorker, см. [здесь](https://doc2.workerman.net/heartbeat.html).

Поскольку UDP соединение является без подключения, то при использовании UDP не будет вызвано обратного вызова onConnect и onClose.

## Параметры функции обратного вызова

 ``` $connection ```

Объект соединения, то есть [экземпляр TcpConnection](../tcp-connection.md), используемый для работы с клиентским соединением, такой как [отправка данных](../tcp-connection/send.md), [закрытие соединения](../tcp-connection/close.md) и т. д.

## Пример

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onClose = function(TcpConnection $connection)
{
    echo "соединение закрыто\n";
};
// Запуск worker
Worker::runAll();
```

Примечание: помимо использования анонимной функции в качестве обратного вызова, можно также [просмотреть здесь](../faq/callback_methods.md) другие способы записи обратных вызовов.
