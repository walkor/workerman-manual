# Корутины Fiber
workerman начиная с версии 5.0.0 поддерживает [корутины Fiber](https://www.php.net/manual/zh/language.fibers.php).

> **Обратите внимание**
> Для корутин требуется PHP>=8.1 и установка `composer require revolt/event-loop ^1.0.0`

### Введение

Fiber - это встроенные в PHP корутины, которые могут прерывать выполнение PHP-кода и восстанавливать его в нужный момент. Основное преимущество заключается в том, что разработчики могут писать асинхронный неблокирующий код синхронным образом, что значительно повышает его поддерживаемость.

### Пример

Давайте сравним использование корутин и асинхронных обратных вызовов с помощью примера. Предположим, у нас есть задача вызвать HTTP-интерфейс, а затем задержать ответ на одну секунду. Примеры асинхронного обратного вызова и использования корутин приведены ниже.

**Использование асинхронных обратных вызовов**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Http\Client;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    static $http;
    $http = $http ?: new Client();
    // Запрос HTTP-интерфейса
    $http->get('http://example.com/', function ($response) use ($connection) {
        // Отправка через секунду
        Timer::add(1, function() use ($connection, $response) {
            // Отправка данных в браузер
            $connection->send((string)$response->getBody());
        }, null, false);
    });
};

Worker::runAll();
```

**Использование корутин**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Http\Client;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    static $http;
    $http = $http ?: new Client();
    // Вызов HTTP-интерфейса
    $response = $http->get('http://example.com/');
    // Задержка на 1 секунду
    Timer::sleep(1);
    // Отправка данных
    $connection->send((string)$response->getBody());
};

Worker::runAll();
```

> **Обратите внимание**
> Для приведенного выше кода требуется установка `composer require workerman/http-client ^2.0.0`.

Оба этих способа асинхронные и неблокирующие, обладают хорошей производительностью, но использование корутин делает код более читаемым и удобным для поддержки.

### Важные моменты при использовании корутин

* Корутины Fiber не поддерживают состояние Pdo, Redis и встроенные PHP блокирующие функции, то есть использование этих расширений и функций все равно вызовет блокировку.
* В настоящее время доступны клиенты корутин Fiber [workerman/http-client](../components/workerman-http-client.md), [workerman/redis](../components/workerman-redis.md)

# Корутины Swoole
workerman v5 также поддерживает использование Swoole в качестве нижележащего событийного драйвера


```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Swoole\Coroutine\Http\Client;
use Swoole\Coroutine;

// Здесь нужно вручную установить Swoole в качестве нижележащего событийного драйвера
Worker::$eventLoopClass = Workerman\Events\Swoole::class;
$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    Coroutine::create(function() use ($connection) {
        $cli = new Client('example.com', 80);
        $cli->get('/get');
        $connection->send($cli->body);
    });
};

Worker::runAll();
```

**Подсказка**
* Рекомендуется использовать Swoole 5.0 или более поздние версии
* Использование Swoole в качестве нижележащего событийного драйвера позволяет workerman поддерживать корутины Swoole.
* При использовании Swoole в качестве нижележащего событийного драйвера не требуется устанавливать модуль event.
* По умолчанию Swoole не активирует корутины одним нажатием, то есть база данных Pdo, Redis, встроенные файловые операции PHP осуществляются блокировкой.
* Для активации корутин необходимо вручную вызвать `\Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_ALL);`

Более подробную информацию смотрите в [Документации по Swoole](https://wiki.swoole.com/)

Для дополнительной информации обратитесь к [Событийному драйверу](appendices/event.md)

# О корутинах
Прежде всего, не стоит излишне верить в корутины. Когда базы данных, Redis и другие хранилища находятся в локальной сети, многопроцессовые блокирующие вызовы часто бывают быстрее, чем корутины. Согласно [данным тестирования с прошествием трех лет на techempower.com](https://www.techempower.com/benchmarks/#section=data-r21&l=zik073-6bj&test=db), производительность многопроцессового вызова базы данных в workerman лучше, чем соединение с пулом соединений и использованием корутин в swoole, и даже превосходит производительность асинхронных фреймворков на go, таких как gin, echo, почти вдвое.

workerman уже повысил производительность приложений на PHP в несколько раз, поэтому в большинстве проектов на workerman использование корутин может не привести к значительному увеличению производительности.
Если в вашей системе есть медленные вызовы, например, внешние HTTP вызовы, стоит рассмотреть использование корутин для увеличения производительности.


