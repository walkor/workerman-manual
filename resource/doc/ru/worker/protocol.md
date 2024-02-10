# протокол
Требования (workerman >= 3.2.7)

## Описание:
```php
string Worker::$protocol
```

Устанавливает класс протокола для текущего экземпляра Worker.

Примечание: класс обработки протокола можно указать непосредственно при инициализации параметров прослушивания Worker, например:

```php
$worker = new Worker('http://0.0.0.0:8686');
```



## Пример


```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8686');
$worker->protocol = 'Workerman\\Protocols\\Http';

$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    $connection->send("hello");
};

// Запуск рабочего процесса
Worker::runAll();
```

Приведенный выше код эквивалентен следующему коду:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

/**
 * Вначале проверяется, есть ли у пользователя пользовательский класс протокола \Protocols\Http,
 * если нет, то используется встроенный класс протокола Workerman\Protocols\Http
 */
$worker = new Worker('http://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    $connection->send("hello");
};

// Запуск рабочего процесса
Worker::runAll();
```
