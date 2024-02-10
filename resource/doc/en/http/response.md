# Introduction

Since version 4.x, Workerman has strengthened its support for HTTP services, introducing request classes, response classes, session classes, and [SSE](SSE.md). If you want to use Workerman's HTTP service, it is highly recommended to use Workerman 4.x or later versions.

**Please note that the following usage is for Workerman 4.x, and is not compatible with Workerman 3.x.**

## Notes

- Unless a chunk or SSE response is being sent, it is not allowed to send multiple responses in one request, which means that multiple calls to `$connection->send()` are not allowed in one request.
- Each request must call `$connection->send()` to send the response, otherwise the client will keep waiting.

## Quick Response

When there is no need to change the HTTP status code (default 200), or customize headers, or cookies, you can directly send a string to the client to complete the response.

**Example**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // Directly send "this is body" to the client
    $connection->send("this is body");
};

// Run the worker
Worker::runAll();
```

## Change Status Code

When custom status code, headers, or cookies are needed, you should use the `Workerman\Protocols\Http\Response` response class. For example, in the following example, a status code of 404 is returned when accessing the path `/404`, with the body content `<h1>Sorry, the file does not exist</h1>`.

**Example**
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
        $connection->send(new Response(404, [], '<h1>Sorry, the file does not exist</h1>'));
    } else {
        $connection->send('this is body');
    }
};

// Run the worker
Worker::runAll();
```
Once the `Response` class is initialized, the following method can be used to change the status code.
```php
$response = new Response(200);
$response->withStatus(404);
$connection->send($response);
```

## Send Headers

Similarly, sending headers requires the use of the `Workerman\Protocols\Http\Response` response class.

**Example**
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
        'X-Header-One' => 'Header Value'
    ], 'this is body');
    $connection->send($response);
};

// Run the worker
Worker::runAll();
```
Once the `Response` class is initialized, the following method can be used to add or change headers.
```php
$response = new Response(200);
// Add or change a header
$response->header('Content-Type', 'text/html');
// Add or change multiple headers
$response->withHeaders([
    'Content-Type' => 'application/ json',
    'X-Header-One' => 'Header Value'
]);
$connection->send($response);
```

## Redirection

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $location = '/test_location';
    $response = new Response(302, ['Location' => $location]);
    $connection->send($response);
};
Worker::runAll();
```

## Send Cookies

Similarly, sending cookies requires the use of the `Workerman\Protocols\Http\Response` response class.

**Example**
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

// Run the worker
Worker::runAll();
```

## Send Files

Similarly, sending files requires the use of the `Workerman\Protocols\Http\Response` response class.

To send a file, use the following method:
```php
$response = (new Response())->withFile($file);
$connection->send($response);
```
- Workerman supports sending very large files.
- For large files (over 2M), Workerman will not read the entire file into memory at once, but will read and send the file in segments at the appropriate time.
- Workerman optimizes file read and send speed based on the client's receiving speed, ensuring the fastest file transmission while minimizing memory usage.
- Data transmission is non-blocking and does not affect the processing of other requests.
- When sending files, Workerman automatically adds the `Last-Modified` header, so that the server can determine whether to send a 304 response next time to save file transmission and improve performance.
- The file sent will automatically use the appropriate `Content-Type` header to send to the browser.
- If the file does not exist, it will automatically be converted to a 404 response.

**Example**
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
    // Check if-modified-since header to determine if the file has been modified
    if (!empty($if_modified_since = $request->header('if-modified-since'))) {
        $modified_time = date('D, d M Y H:i:s',  filemtime($file)) . ' ' . \date_default_timezone_get();
        // Return 304 if the file has not been modified
        if ($modified_time === $if_modified_since) {
            $connection->send(new Response(304));
            return;
        }
    }
    // Send the file if it has been modified or there is no if-modified-since header
    $response = (new Response())->withFile($file);
    $connection->send($response);
};

// Run the worker
Worker::runAll();
```

## Send HTTP Chunk Data

- First, you must send a Response response with the `Transfer-Encoding: chunked` header to the client.
- Use the `Workerman\Protocols\Http\Chunk` class to send subsequent chunk data.
- Finally, you must send an empty chunk to end the response.

**Example**
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
    // First send a Response with the Transfer-Encoding: chunked header
    $connection->send(new Response(200, array('Transfer-Encoding' => 'chunked'), 'hello'));
    // Use the Workerman\Protocols\Http\Chunk class to send subsequent Chunk data
    $connection->send(new Chunk('First chunk of data'));
    $connection->send(new Chunk('Second chunk of data'));
    $connection->send(new Chunk('Third chunk of data'));
    // Finally, send an empty chunk to end the response
    $connection->send(new Chunk(''));
};

// Run the worker
Worker::runAll();
```
