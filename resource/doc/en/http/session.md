# Introduction

Since version 4.x, workerman has enhanced support for HTTP services. It introduces request class, response class, session class, and [SSE](SSE.md). If you want to use workerman's HTTP service, it is strongly recommended to use workerman 4.x or later versions.

**Note that the following usages are for workerman 4.x, and are not compatible with workerman 3.x.**


# Get the session object
```php
$session = $request->session();
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
    $session = $request->session();
    $session->set('name', 'tome');
    $connection->send($session->get('name'));
};

// Run the worker
Worker::runAll();
```
**Note**
- The session must be manipulated before the `$connection->send()` method is called.
- The session will automatically save the changes when the object is destroyed, so do not save the object returned by `$request->session()` in a global array or class member, which may prevent the session from being saved.
- By default, the session is stored in a disk file, but it is recommended to use redis for better performance.


## Get all session data
```php
$session = $request->session();
$all = $session->all();
```
It returns an array. If there is no session data, it returns an empty array.

**Example**
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

// Run the worker
Worker::runAll();
```


## Get a value from the session
```php
$session = $request->session();
$name = $session->get('name');
```
It returns null if the data does not exist.

You can also pass a default value as the second parameter to the get method, which will be returned if the corresponding value is not found in the session array. For example:
```php
$session = $request->session();
$name = $session->get('name', 'tom');
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
    $session = $request->session();
    $connection->send($session->get('name', 'tom'));
};

// Run the worker
Worker::runAll();
```

## Store session data
Use the `set` method to store a piece of data.
```php
$session = $request->session();
$session->set('name', 'tom');
```
The `set` method has no return value, and the session will be automatically saved when the object is destroyed.

When storing multiple values, use the `put` method.
```php
$session = $request->session();
$session->put(['name' => 'tom', 'age' => 12]);
```
Similarly, the `put` method has no return value.

**Example**
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

// Run the worker
Worker::runAll();
```

## Delete session data
Use the `forget` method to delete one or more session data.
```php
$session = $request->session();
// Delete one item
$session->forget('name');
// Delete multiple items
$session->forget(['name', 'age']);
```

Additionally, the system provides a `delete` method, which differs from `forget` in that `delete` can only delete one item.
```php
$session = $request->session();
// Equivalent to $session->forget('name');
$session->delete('name');
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
    $request->session()->forget('name');
    $connection->send('ok');
};

// Run the worker
Worker::runAll();
```

## Get and delete a value from the session
```php
$session = $request->session();
$name = $session->pull('name');
```
It has the same effect as the following code:
```php
$session = $request->session();
$value = $session->get($name);
$session->delete($name);
```
It returns null if the corresponding session does not exist.

**Example**
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

// Run the worker
Worker::runAll();
```

## Delete all session data
```php
$request->session()->flush();
```
There is no return value, and the session will be automatically removed from storage when the object is destroyed.

**Example**
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

// Run the worker
Worker::runAll();
```

## Determine if specific session data exists
```php
$session = $request->session();
$has = $session->has('name');
```
The above returns false if the corresponding session does not exist or the session value is null; otherwise, it returns true.

```php
$session = $request->session();
$has = $session->exists('name');
```
The above code is also used to determine if the session data exists, but it returns true even if the corresponding session item value is null.
