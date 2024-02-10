# Introduction
Since version 4.x, Workerman has enhanced its support for HTTP services. It has introduced request, response, session classes and Server-Sent Events (SSE). If you want to use Workerman's HTTP service, it is strongly recommended to use Workerman version 4.x or later.

**Note that all the following usage is applicable for Workerman 4.x and is not compatible with Workerman 3.x.**

## Obtaining the Request Object
The request object is always obtained in the `onMessage` callback function, and the framework will automatically pass the Request object as the second parameter to the callback function.

**Example**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // $request is the request object, and here no operation is performed on the request object, and 'hello' is directly returned to the browser
    $connection->send("hello");
};

// Run the worker
Worker::runAll();
```

When accessing `http://127.0.0.1:8080` from the browser, it will return `hello`.

## Getting GET Request Parameters
**Get the entire GET array**
```php
$get = $request->get();
```
If the request does not have GET parameters, it returns an empty array.

**Get a specific value from the GET array**
```php
$name = $request->get('name');
```
If the GET array does not contain this value, it returns null.

You can also pass a default value as the second parameter to the get method. If the corresponding value is not found in the GET array, it returns the default value. For example:
```php
$name = $request->get('name', 'tom');
```

**Example**
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

// Run the worker
Worker::runAll();
```

When accessing `http://127.0.0.1:8080?name=jerry&age=12` from the browser, it will return `jerry`.

## Getting POST Request Parameters
**Get the entire POST array**
```php
$post = $request->post();
```
If the request does not have POST parameters, it returns an empty array.

**Get a specific value from the POST array**
```php
$name = $request->post('name');
```
If the POST array does not contain this value, it returns null.

Similar to the get method, you can also pass a default value as the second parameter to the post method. If the corresponding value is not found in the POST array, it returns the default value. For example:
```php
$name = $request->post('name', 'tom');
```

**Example**
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

// Run the worker
Worker::runAll();
```
## Get request method
```php
$method = $request->method();
```
The return value could be one of `GET`, `POST`, `PUT`, `DELETE`, `OPTIONS`, or `HEAD`.

**Example**
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

// Run worker
Worker::runAll();
```
## Get request uri
```php
$uri = $request->uri();
```
Returns the requested uri, including path and queryString parts.

**Example**
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

// Run worker
Worker::runAll();
```
When accessing `http://127.0.0.1:8080/user/get.php?uid=10&type=2`, it will return `/user/get.php?uid=10&type=2`.

## Get request path
```php
$path = $request->path();
```
Returns the path part of the request.

**Example**
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

// Run worker
Worker::runAll();
```
When accessing `http://127.0.0.1:8080/user/get.php?uid=10&type=2`, it will return `/user/get.php`.

## Get request queryString
```php
$query_string = $request->queryString();
```
Returns the queryString part of the request.

**Example**
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

// Run worker
Worker::runAll();
```
When accessing `http://127.0.0.1:8080/user/get.php?uid=10&type=2`, it will return `uid=10&type=2`.

## Get request HTTP version
```php
$version = $request->protocolVersion();
```
Returns the string `1.1` or `1.0`.

**Example**
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

// Run worker
Worker::runAll();
```

## Get request sessionId
```php
$sid = $request->sessionId();
```
Returns a string composed of letters and numbers.

**Example**
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

// Run worker
Worker::runAll();
```
