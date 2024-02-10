# Объяснение

С версии 4.x Workerman улучшил поддержку HTTP-сервиса. Были введены классы запроса, ответа, сессии, а также [SSE](SSE.md). Если вы хотите использовать HTTP-сервис Workerman, настоятельно рекомендуется использовать версию 4.x или более поздние.

**Обратите внимание, что ниже приведены примеры использования Workerman версии 4.x, несовместимые с Workerman 3.x.**

# Примечание

- За исключением ответа в формате chunk или SSE, не допускается многократная отправка ответа в одном запросе, то есть не разрешается многократный вызов `$connection->send()`.
- Необходимо вызвать `$connection->send()` один раз для каждого запроса, иначе клиент будет бесконечно ожидать.

## Быстрый ответ
Если необходимо изменить HTTP-статусный код (по умолчанию 200), или настроить заголовок, cookie, можно напрямую отправить строку в ответ клиенту.

**Пример**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // Отправить клиенту сообщение "this is body"
    $connection->send("this is body");
};

// Запуск воркера
Worker::runAll();
``` 

## Изменение статусного кода
Если нужно настроить статусный код, заголовок, cookie, необходимо использовать класс ответа `Workerman\Protocols\Http\Response`. Например, в следующем примере при обращении к пути `/404` будет возвращен статусный код 404, а в теле ответа будет содержаться текст "<h1>Извините, файл не найден</h1>".

**Пример**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    if ($request->path() === '/404') {
        $connection->send(new Response(404, [], '<h1>Извините, файл не найден</h1>'));
    } else {
        $connection->send('this is body');
    }
};

// Запуск воркера
Worker::runAll();
```
После инициализации класса `Response` для изменения статусного кода можно использовать следующий метод.
```php
$response = new Response(200);
$response->withStatus(404);
$connection->send($response);
```

## Отправка заголовков
Также для отправки заголовков необходимо использовать класс ответа `Workerman\Protocols\Http\Response`.

**Пример**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $response = new Response(200, [
        'Content-Type' => 'text/html',
        'X-Header-One' => 'Значение заголовка'
    ], 'this is body');
    $connection->send($response);
};

// Запуск воркера
Worker::runAll();
```
После инициализации класса `Response` для добавления или изменения заголовков можно использовать следующий метод.
```php
$response = new Response(200);
// Добавить или изменить один заголовок
$response->header('Content-Type', 'text/html');
// Добавить или изменить несколько заголовков
$response->withHeaders([
    'Content-Type' => 'application/json',
    'X-Header-One' => 'Значение заголовка'
]);
$connection->send($response);
```

## Перенаправление
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)

$worker = new Worker('http://0.0.0.0:8080');
$worker->onMessage = function($connection, $request)
{
    $location = '/test_location';
    $response = new Response(302, ['Location' => $location]);
    $connection->send($response);
};
Worker::runAll();
```

## Отправка cookie
Также для отправки cookie необходимо использовать класс ответа `Workerman\Protocols\Http\Response`.

**Пример**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $response = new Response(200, [], 'this is body');
    $response->cookie('name', 'tom');
    $connection->send($response);
};

// Запуск воркера
Worker::runAll();
```

## Отправка файла
Также для отправки файлов необходимо использовать класс ответа `Workerman\Protocols\Http\Response`.

Для отправки файла используйте следующую конструкцию
```php
$response = (new Response())->withFile($file);
$connection->send($response);
```
- Workerman поддерживает отправку очень больших файлов
- Для больших файлов (более 2МБ) Workerman не считывает весь файл целиком в память, а вместо этого выбирает правильный момент для поэтапного считывания и отправки файла.
- Workerman оптимизирует скорость отправки файла в зависимости от скорости приема клиентом, обеспечивая максимальную скорость отправки файла при минимальном использовании памяти.
- Передача данных не блокируется и не влияет на обработку других запросов
- При отправке файла автоматически добавляется заголовок `Last-Modified`, чтобы сервер мог в следующий раз определить, отправлять ли ответ 304, сэкономив трафик и повысив производительность.
- Отправляемый файл автоматически отправляется с правильным заголовком `Content-Type` для браузера
- Если файла не существует, он автоматически отправляется как ответ 404

**Пример**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $file = '/your/path/of/file';
    // Проверка заголовка if-modified-since для определения, менялся ли файл
    if (!empty($if_modified_since = $request->header('if-modified-since'))) {
        $modified_time = date('D, d M Y H:i:s', filemtime($file)) . ' ' . \date_default_timezone_get();
        // Если файл не изменен, возвратить 304
        if ($modified_time === $if_modified_since) {
            $connection->send(new Response(304));
            return;
        }
    }
    // Если файл изменен или заголовок if-modified-since отсутствует, отправить файл
    $response = (new Response())->withFile($file);
    $connection->send($response);
};

// Запуск воркера
Worker::runAll();
```

## Отправка данных в формате http chunk
 - Сначала необходимо отправить ответ `Response` с заголовком `Transfer-Encoding: chunked` клиенту
 - Далее для отправки дополнительных chunk данных используйте класс `Workerman\Protocols\Http\Chunk`
 - В конце необходимо отправить пустой chunk, чтобы завершить ответ

**Пример**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
use Workerman\Protocols\Http\Chunk;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // Сначала отправить ответ с заголовком Transfer-Encoding: chunked
    $connection->send(new Response(200, array('Transfer-Encoding' => 'chunked'), 'hello'));
    // Далее для отправки дополнительных chunk данных используйте класс Workerman\Protocols\Http\Chunk
    $connection->send(new Chunk('Первый сегмент данных'));
    $connection->send(new Chunk('Второй сегмент данных'));
    $connection->send(new Chunk('Третий сегмент данных'));
   //  В конце обязательно отправить пустой chunk, чтобы завершить ответ
    $connection->send(new Chunk(''));
};

// Запуск воркера
Worker::runAll();
```
