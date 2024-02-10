# Метод reConnect

```php
void AsyncTcpConnection::reConnect(float $delay = 0)
```

``` (требуется версия Workerman >= 3.3.5) ```

Переподключение. Обычно вызывается в обратном вызове ```onClose```, чтобы реализовать повторное подключение после разрыва соединения.

Если соединение разорвано из-за проблем с сетью или перезапуска службы у удаленной стороны, то можно вызвать этот метод для повторного подключения.

### Параметры
``` $delay ```

Задержка перед выполнением повторного подключения. Выражается в секундах, поддерживает дробные значения, можно указать точность до миллисекунд.

Если параметр пропущен или равен 0, то подразумевается мгновенное повторное подключение.

Рекомендуется передавать параметр для отложенного выполнения повторного подключения, чтобы избежать избыточной нагрузки на процессор из-за бесконечных попыток подключения из-за проблем с удаленной службой.

### Возвращаемые значения
Нет возвращаемого значения

### Пример

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker)
{
    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };
    $con->onMessage = function(AsyncTcpConnection $con, $msg) {
        echo "recv $msg\n";
    };
    $con->onClose = function(AsyncTcpConnection $con) {
        // Если соединение разорвано, то повторное подключение через 1 секунду
        $con->reConnect(1);
    };
    $con->connect();
};

Worker::runAll();
```

> **Примечание**
> После успешного повторного подключения метод onConnect $con будет вызван снова (если он установлен). Иногда нам нужно, чтобы onConnect выполнился только один раз, а при повторном подключении не вызвался. См. пример ниже:

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker)
{
    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    $con->onConnect = function(AsyncTcpConnection $con) {
        static $is_first_connect = true;
        if (!$is_first_connect) return;
        $is_first_connect = false;
        $con->send('hello');
    };
    $con->onMessage = function(AsyncTcpConnection $con, $msg) {
        echo "recv $msg\n";
    };
    $con->onClose = function(AsyncTcpConnection $con) {
        // Если соединение разорвано, то повторное подключение через 1 секунду
        $con->reConnect(1);
    };
    $con->connect();
};

Worker::runAll();
```
