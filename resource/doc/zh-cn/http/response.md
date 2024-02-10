# 说明

workerman从4.x版本开始加强了HTTP服务的支持。引入了请求类、响应类、session类以及[SSE](SSE.md)。如果你想使用workerman的HTTP服务，强烈推荐使用workerman4.x或者以后的更高版本。

**注意以下都是workerman4.x版本的用法，不兼容workerman3.x。**


# 注意

 - 除非发送的是chunk或者SSE响应，否则不允许在一个请求里多次发送响应，也就是在一个请求里不允许多次调用`$connection->send()`。
 - 每个请求最终都需要调用一次`$connection->send()`发送响应，否则客户端会一直等待

##  快捷响应
当不需要更改HTTP状态码(默认200)，或者自定义header、cookie时，可以直接向客户端发送字符串完成响应。

**例子**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // 直接发送this is body给客户端
    $connection->send("this is body");
};

// 运行worker
Worker::runAll();
```

##  更改状态码
当需要自定义状态码、header、cookie时，需要使用`Workerman\Protocols\Http\Response`响应类。例如下面例子在访问路径为`/404`时返回404的状态码，包体内容为`<h1>抱歉，文件不存在</h1>`。

**例子**
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
        $connection->send(new Response(404, [], '<h1>抱歉，文件不存在</h1>'));
    } else {
        $connection->send('this is body');
    }
};

// 运行worker
Worker::runAll();
```
当`Response`类已经初始化后，想更改状态码使用下面方法。
```php
$response = new Response(200);
$response->withStatus(404);
$connection->send($response);
```

## 发送header
同样的，发送header需要使用`Workerman\Protocols\Http\Response`响应类。

**例子**
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

// 运行worker
Worker::runAll();
```
当`Response`类已经初始化后，想增加或者更改header使用下面方法。
```php
$response = new Response(200);
// 添加或者更改一个header
$response->header('Content-Type', 'text/html');
// 添加或者更改多个header
$response->withHeaders([
    'Content-Type' => 'application/ json',
    'X-Header-One' => 'Header Value'
]);
$connection->send($response);
```

## 重定向
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

## 发送cookie
同样的，发送cookie需要使用`Workerman\Protocols\Http\Response`响应类。

**例子**
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

// 运行worker
Worker::runAll();
```

## 发送文件
同样的，发送文件需要使用`Workerman\Protocols\Http\Response`响应类。

发送文件时用以下方式
```php
$response = (new Response())->withFile($file);
$connection->send($response);
```
 - workerman支持发送超大文件
 - 对于大文件(超过2M)，workerman不会将整个文件一次性读入内存，而是在合适的时机分段读取文件并发送
 - workerman会根据客户端接收速度来优化文件读取发送速度，保证最快速发送文件的同时将内存占用减少到最低
 - 数据发送是非阻塞的，不会影响其它请求处理
 - 发送文件时会自动加上`Last-Modified`头，以便下次请求时服务端判断是否发送304响应以节省文件传输提高性能
 - 发送的文件会自动使用合适的`Content-Type`头发送给浏览器
 - 如果文件不存在，会自动转为404响应

**例子**
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
    // 检查if-modified-since头判断文件是否修改过
    if (!empty($if_modified_since = $request->header('if-modified-since'))) {
        $modified_time = date('D, d M Y H:i:s',  filemtime($file)) . ' ' . \date_default_timezone_get();
        // 文件未修改则返回304
        if ($modified_time === $if_modified_since) {
            $connection->send(new Response(304));
            return;
        }
    }
    // 文件修改过或者没有if-modified-since头则发送文件
    $response = (new Response())->withFile($file);
    $connection->send($response);
};

// 运行worker
Worker::runAll();
```

## 发送http chunk数据
 - 必须先发送一个携带 `Transfer-Encoding: chunked`头的Response响应给客户端
 - 发送后续chunk数据使用`Workerman\Protocols\Http\Chunk` 类
 - 最终必须发送一个空的chunk来结束响应

**例子**
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
    // 首先发送一个带Transfer-Encoding: chunked头的Response响应
    $connection->send(new Response(200, array('Transfer-Encoding' => 'chunked'), 'hello'));
    // 后续Chunk数据用Workerman\Protocols\Http\Chunk类发送
    $connection->send(new Chunk('第一段数据'));
    $connection->send(new Chunk('第二段数据'));
    $connection->send(new Chunk('第三段数据'));
   //  最后必须发送一个空的chunk结束响应
    $connection->send(new Chunk(''));
};

// 运行worker
Worker::runAll();
```

