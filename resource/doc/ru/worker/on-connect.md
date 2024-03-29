# onConnect
## Описание:
```php
callback Worker::$onConnect
```

Вызывается обратный вызов при установлении соединения клиента с Workerman (после завершения трехстороннего TCP-рукопожатия). Каждое соединение вызывает обратный вызов `onConnect` только один раз.

Примечание: событие onConnect представляет только установление TCP-соединения клиента с Workerman; в это время клиент еще не отправил никаких данных. Поэтому, кроме получения IP-адреса с помощью `$connection->getRemoteIp()`, в onConnect нет другой информации о клиенте, и нельзя определить, кто является клиентом. Чтобы узнать, кто является клиентом, необходимо, чтобы клиент отправил аутентификационные данные, такие как токен или имя пользователя и пароль, и выполнить аутентификацию в [обратном вызове onMessage](on-message.md).

Поскольку UDP является безсоединительным протоколом, при использовании UDP обратный вызов onConnect не будет вызван, также как и обратный вызов onClose.

## Параметры обратного вызова

``` $connection ```

Объект соединения, то есть [экземпляр TcpConnection](../tcp-connection.md), используемый для работы с клиентским соединением, таких как [отправка данных](../tcp-connection/send.md), [закрытие соединения](../tcp-connection/close.md) и т. д.

## Пример

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "новое соединение с IP " . $connection->getRemoteIp() . "\n";
};
// Запуск worker
Worker::runAll();
```

Примечание: помимо использования анонимных функций в качестве обратного вызова, можно также [посмотреть здесь](../faq/callback_methods.md) другие способы написания обратных вызовов.
