# Инструкция

Начиная с версии 4.x Workerman улучшил поддержку HTTP-сервисов. Был введен класс запроса, класс ответа, класс сеансов, а также [SSE](SSE.md). Если вы хотите использовать HTTP-сервисы Workerman, настоятельно рекомендуется использовать версию 4.x или более поздние.

**Обратите внимание, что все примеры даны для версии Workerman 4.x и не совместимы с версией Workerman 3.x.**

# Получение объекта сеанса
```php
$session = $request->session();
```

**Пример**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('name', 'tome');
    $connection->send($session->get('name'));
};

// Запуск worker
Worker::runAll();
```
**Примечания**
- Сеанс должен быть обработан перед вызовом `$connection->send()`.
- Сеанс автоматически сохраняется при уничтожении объекта, поэтому не следует хранить объект, возвращенный методом `$request->session()`, в глобальном массиве или как член класса, что может привести к невозможности сохранения сеанса.
- По умолчанию сеанс хранится в файловой системе, для более высокой производительности рекомендуется использовать Redis.

## Получение всех данных сеанса
```php
$session = $request->session();
$all = $session->all();
```
Возвращает массив. Если данных сеанса нет, возвращает пустой массив.

**Пример**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('name', 'tom');
    $connection->send(var_export($session->all(), true));
};

// Запуск worker
Worker::runAll();
```

## Получение значения из сеанса
```php
$session = $request->session();
$name = $session->get('name');
```
Если данных нет, возвращает null.

Также можно передать второй аргумент метода get в качестве значения по умолчанию, которое вернется, если значение не будет найдено в массиве сеанса. Например:
```php
$session = $request->session();
$name = $session->get('name', 'tom');
```
**Пример**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $connection->send($session->get('name', 'tom'));
};

// Запуск worker
Worker::runAll();
```

## Сохранение данных сеанса
Для этого используется метод set при сохранении отдельного значения.
```php
$session = $request->session();
$session->set('name', 'tom');
```
Метод set не возвращает значения, сеанс автоматически сохраняется при удалении объекта.

При сохранении нескольких значений используется метод put.
```php
$session = $request->session();
$session->put(['name' => 'tom', 'age' => 12]);
```
Также метод put не возвращает значения.

**Пример**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('name', 'tom');
    $connection->send($session->get('name'));
};

// Запуск worker
Worker::runAll();
```

## Удаление данных сеанса
Для удаления отдельного или нескольких значений сеанса используется метод `forget`.
```php
$session = $request->session();
// Удаление одного значения
$session->forget('name');
// Удаление нескольких значений
$session->forget(['name', 'age']);
```

Кроме того, существует метод `delete`, который отличается от метода `forget` тем, что может удалить только одно значение.
```php
$session = $request->session();
// Эквивалентно $session->forget('name');
$session->delete('name');
```

**Пример**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $request->session()->forget('name');
    $connection->send('ok');
};

// Запуск worker
Worker::runAll();
```

## Получение и удаление значения из сеанса
```php
$session = $request->session();
$name = $session->pull('name');
```
Эффект такой же, как и в следующем коде
```php
$session = $request->session();
$value = $session->get($name);
$session->delete($name);
```
Если соответствующего сеанса не существует, возвращает null.
 
**Пример**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->session()->pull('name'));
};

// Запуск worker
Worker::runAll();
```

## Удаление всех данных сеанса
```php
$request->session()->flush();
```
Метод не возвращает значения, сеанс автоматически удаляется из хранилища при удалении объекта.

**Пример**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $request->session()->flush();
    $connection->send('ok');
};

// Запуск worker
Worker::runAll();
```

## Проверка наличия данных в сеансе
```php
$session = $request->session();
$has = $session->has('name');
```
Если соответствующий сеанс отсутствует или значение соответствующего сеанса равно null, возвращает false, в противном случае возвращает true.

```php
$session = $request->session();
$has = $session->exists('name');
```
Этот код также используется для проверки наличия данных в сеансе, однако, в отличие от первого метода, если значение соответствующего сеанса равно null, он также возвращает true.
