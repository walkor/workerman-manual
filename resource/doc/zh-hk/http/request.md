# 說明
Workerman從4.x版本開始加強了HTTP服務的支持。引入了請求類、響應類、session類以及[SSE](SSE.md)。如果你想使用Workerman的HTTP服務，強烈推薦使用Workerman 4.x或者以後的更高版本。

**注意以下都是Workerman 4.x版本的用法，不相容Workerman 3.x。**

## 獲得請求對象
請求對象一律在onMessage回調函數中獲取，框架會自動將Request對象通過回調函數第二個參數傳遞進來。

**例子**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // $request炲請求對象，這裡沒有對請求對象執行任何操作直接返回hello給瀏覽器
    $connection->send("hello");
};

// 運行worker
Worker::runAll();
```

當瀏覽器訪問`http://127.0.0.1:8080`時將返回`hello`。

## 獲得請求參數get

**獲取整個get陣列**
```php
$get = $request->get();
```
如果請求沒有get參數則返回一個空的陣列。

**獲取get陣列的某一個值**
```php
$name = $request->get('name');
```
如果get陣列中不包含這個值則返回null。

你也可以給get方法第二個參數傳遞一個默認值，如果get陣列中沒找到對應值則返回默認值。例如：
```php
$name = $request->get('name', 'tom');
```

**例子**
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

// 運行worker
Worker::runAll();
```

當瀏覽器訪問`http://127.0.0.1:8080?name=jerry&age=12`時將返回`jerry`。

## 獲取請求參數post
**獲取整個post陣列**
```php
$post = $request->post();
```
如果請求沒有post參數則返回一個空的陣列。

**獲取post陣列的某一個值**
```php
$name = $request->post('name');
```
如果post陣列中不包含這個值則返回null。

與get方法一樣，你也可以給post方法第二個參數傳遞一個默認值，如果post陣列中沒找到對應值則返回默認值。例如：
```php
$name = $request->post('name', 'tom');
```
**例子**
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

// 運行worker
Worker::runAll();
```

## 獲取原始請求post包體
```php
$post = $request->rawBody();
```
這個功能類似與 `php-fpm`裡的 `file_get_contents("php://input");`操作。用於獲得http原始請求包體。這在獲取非`application/x-www-form-urlencoded`格式的post請求數據時很有用。 

**例子**
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

// 運行worker
Worker::runAll();
```

## 獲取header
**獲取整個header陣列**
```php
$headers = $request->header();
```
如果請求沒有header參數則返回一個空的陣列。注意所有key均為小寫。

**獲取header陣列的某一個值**
```php
$host = $request->header('host');
```
如果header陣列中不包含這個值則返回null。注意所有key均為小寫。

與get方法一樣，你也可以給header方法第二個參數傳遞一個默認值，如果header陣列中沒找到對應值則返回默認值。例如：
```php
$host = $request->header('host', 'localhost');
```
**例子**
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

// 運行worker
Worker::runAll();
```
## 獲取請求方法
```php
$method = $request->method();
```
返回值可能是`GET`、`POST`、`PUT`、`DELETE`、`OPTIONS`、`HEAD`中的一個。

**例子**
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

// 運行worker
Worker::runAll();
```
## 獲取請求uri
```php
$uri = $request->uri();
```
返回請求的uri，包括path和queryString部分。

**例子**
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

// 運行worker
Worker::runAll();
```
當瀏覽器訪問`http://127.0.0.1:8080/user/get.php?uid=10&type=2`時將返回`/user/get.php?uid=10&type=2`。

## 獲取請求路徑

```php
$path = $request->path();
```
返回請求的path部分。

**例子**
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

// 運行worker
Worker::runAll();
```
當瀏覽器訪問`http://127.0.0.1:8080/user/get.php?uid=10&type=2`時將返回`/user/get.php`。

## 獲取請求queryString

```php
$query_string = $request->queryString();
```
返回請求的queryString部分。

**例子**
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

// 運行worker
Worker::runAll();
```
當瀏覽器訪問`http://127.0.0.1:8080/user/get.php?uid=10&type=2`時將返回`uid=10&type=2`。

## 獲取請求HTTP版本

```php
$version = $request->protocolVersion();
```
返回字符串 `1.1` 或者`1.0`。

**例子**
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

// 運行worker
Worker::runAll();
```

## 獲取請求sessionId

```php
$sid = $request->sessionId();
```
返回字符串，由字母和數字組成

**例子**
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

// 運行worker
Worker::runAll();
```
