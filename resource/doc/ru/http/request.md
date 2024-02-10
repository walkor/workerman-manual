# Инструкция
С версии 4.x workerman улучшил поддержку HTTP-сервиса. Были добавлены классы запроса, ответа, сессии, а также [SSE](SSE.md). Если вы хотите использовать HTTP-сервис workerman, настоятельно рекомендуется использовать версию 4.x или более поздние.

**Обратите внимание, что следующие примеры базируются на версии workerman 4.x и несовместимы с workerman 3.x.**

## Получение объекта запроса
Объект запроса всегда получается в функции обратного вызова onMessage. Фреймворк автоматически передает объект Request через второй аргумент функции обратного вызова.

**Пример**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // $request - объект запроса, здесь не производится никаких операций с объектом запроса, просто возвращается "hello" браузеру
    $connection->send("hello");
};

// Запуск Worker
Worker::runAll();
```

При обращении браузера к `http://127.0.0.1:8080` будет возвращено `hello`.

## Получение параметров запроса GET

**Получение всего массива get-параметров**
```php
$get = $request->get();
```
Если запрос не содержит параметров GET, то возвращается пустой массив.

**Получение значения из массива GET**
```php
$name = $request->get('name');
```
Если в массиве GET нет данного значения, то возвращается null.

Также можно передать второй аргумент метода get в качестве значения по умолчанию. Если соответствующее значение в массиве GET не найдено, то будет возвращено значение по умолчанию. Например:
```php
$name = $request->get('name', 'tom');
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
    $connection->send($request->get('name'));
};

// Запуск Worker
Worker::runAll();
```

При обращении браузера к `http://127.0.0.1:8080?name=jerry&age=12` будет возвращено `jerry`.

## Получение параметров запроса POST

**Получение всего массива post-параметров**
```php
$post = $request->post();
```
Если запрос не содержит параметров POST, то возвращается пустой массив.

**Получение значения из массива POST**
```php
$name = $request->post('name');
```
Если в массиве POST нет данного значения, то возвращается null.

Как и в методе get, можно передать второй аргумент метода post в качестве значения по умолчанию. Если соответствующее значение в массиве POST не найдено, то будет возвращено значение по умолчанию. Например:
```php
$name = $request->post('name', 'tom');
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
    $post = $request->post();
    $connection->send(var_export($post, true));
};

// Запуск Worker
Worker::runAll();
```

## Получение исходного содержимого тела запроса POST
```php
$post = $request->rawBody();
```
Эта функция похожа на операцию `file_get_contents("php://input");` в `php-fpm` и используется для получения исходного тела HTTP-запроса. Она полезна при получении данных POST-запросов в формате, отличном от `application/x-www-form-urlencoded`. 

**Пример**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $post = json_decode($request->rawBody());
    $connection->send('hello');
};

// Запуск Worker
Worker::runAll();
```

## Получение заголовков
**Получение всего массива заголовков**
```php
$headers = $request->header();
```
Если запрос не содержит заголовков, то возвращается пустой массив. Обратите внимание, что все ключи являются строчными буквами.

**Получение значения из массива заголовков**
```php
$host = $request->header('host');
```
Если в массиве заголовков нет данного значения, то возвращается null. Обратите внимание, что все ключи являются строчными буквами.

Как и в методах get и post, можно передать второй аргумент метода header в качестве значения по умолчанию. Если соответствующее значение в массиве заголовков не найдено, то будет возвращено значение по умолчанию. Например:
```php
$host = $request->header('host', 'localhost');
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
    if ($request->header('connection') === 'keep-alive') {
        $connection->send('hello');
    } else {
        $connection->close('hello');
    }    
};

// Запуск Worker
Worker::runAll();
```

## Получение cookie
**Получение всего массива cookie**
```php
$cookies = $request->cookie();
```
Если запрос не содержит cookie-параметров, то возвращается пустой массив.

**Получение значения из массива cookie**
```php
$name = $request->cookie('name');
```
Если в массиве cookie нет данного значения, то возвращается null.

Как и в методах get и post, можно передать второй аргумент метода cookie в качестве значения по умолчанию. Если соответствующее значение в массиве cookie не найдено, то будет возвращено значение по умолчанию. Например:
```php
$name = $request->cookie('name', 'tom');
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
    $cookie = $request->cookie();
    $connection->send(var_export($cookie, true));
};

// Запуск Worker
Worker::runAll();
```

## Получение загруженных файлов
**Получение всего массива загруженных файлов**
```php
$files = $request->file();
```
Возвращаемый формат файла аналогичен:
```php
array (
    'avatar' => array (
            'name' => '123.jpg',
            'tmp_name' => '/tmp/workerman.upload.9hjR4w',
            'size' => 1196127,
            'error' => 0,
            'type' => 'application/octet-stream',
      ),
     'anotherfile' =>  array (
            'name' => '456.txt',
            'tmp_name' => '/tmp/workerman.upload.9sirSws',
            'size' => 490,
            'error' => 0,
            'type' => 'text/plain',
      )
)
```
Где:

 - name - название файла
 - tmp_name - временное расположение файла на диске
 - size - размер файла
 - error - [код ошибки](https://www.php.net/manual/zh/features.file-upload.errors.php)
 - type - тип MIME-файла.

**Примечания:**

 - Размер загружаемого файла ограничивается [defaultMaxPackageSize](../tcp-connection/default-max-package-size.md), по умолчанию 10 Мб, но может быть изменен.

 - После завершения запроса файлы будут автоматически удалены.

 - Если в запросе отсутствуют загруженные файлы, вернется пустой массив.

### Получение конкретного загруженного файла
```php
$avatar_file = $request->file('avatar');
```
Возвращается что-то похожее на:
```php
array (
        'name' => '123.jpg',
        'tmp_name' => '/tmp/workerman.upload.9hjR4w',
        'size' => 1196127,
        'error' => 0,
        'type' => 'application/octet-stream',
  )
```
Если файл не найден, вернется null.

**Пример**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $file = $request->file('avatar');
    if ($file && $file['error'] === UPLOAD_ERR_OK) {
        rename($file['tmp_name'], '/home/www/web/public/123.jpg');
        $connection->send('ok');
        return;
    }
    $connection->send('upload fail');
};

// Запуск Worker
Worker::runAll();
```
## Получение хоста
Получение информации о хосте запроса.
```php
$host = $request->host();
```
Если адрес запроса нестандартный и не соответствует портам 80 или 443, информация о хосте может содержать порт, например `example.com:8080`. Если порт не нужен, первым параметром можно передать `true`.
```php
$host = $request->host(true);
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
    $connection->send($request->host());
};

// Запуск воркера
Worker::runAll();
```
При обращении браузера по адресу `http://127.0.0.1:8080?name=tom` будет возвращено `127.0.0.1:8080`.

## Получение метода запроса
```php
$method = $request->method();
```
Возвращает одно из значений `GET`, `POST`, `PUT`, `DELETE`, `OPTIONS`, `HEAD`.

**Пример**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->method());
};

// Запуск воркера
Worker::runAll();
```
## Получение URI запроса
```php
$uri = $request->uri();
```
Возвращает запрошенный URI, включая путь и строку запроса.

**Пример**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->uri());
};

// Запуск воркера
Worker::runAll();
```
При обращении браузера по адресу `http://127.0.0.1:8080/user/get.php?uid=10&type=2` будет возвращено `/user/get.php?uid=10&type=2`.

## Получение пути запроса
```php
$path = $request->path();
```
Возвращает запрошенный путь.

**Пример**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->path());
};

// Запуск воркера
Worker::runAll();
```
При обращении браузера по адресу `http://127.0.0.1:8080/user/get.php?uid=10&type=2` будет возвращено `/user/get.php`.

## Получение строки запроса
```php
$query_string = $request->queryString();
```
Возвращает строку запроса.

**Пример**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->queryString());
};

// Запуск воркера
Worker::runAll();
```
При обращении браузера по адресу `http://127.0.0.1:8080/user/get.php?uid=10&type=2` будет возвращено `uid=10&type=2`.

## Получение версии HTTP протокола запроса
```php
$version = $request->protocolVersion();
```
Возвращает строку `1.1` или `1.0`.

**Пример**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->protocolVersion());
};

// Запуск воркера
Worker::runAll();
```

## Получение идентификатора сеанса запроса
```php
$sid = $request->sessionId();
```
Возвращает строку, состоящую из букв и цифр.

**Пример**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->sessionId());
};

// Запуск воркера
Worker::runAll();
```
