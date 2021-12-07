# 说明
workerman从4.x版本开始加强了HTTP服务的支持。引入了请求类、响应类、session类以及[SSE](SSE.md)。如果你想使用workerman的HTTP服务，强烈推荐使用workerman4.x或者以后的更高版本。

**注意以下都是workerman4.x版本的用法，不兼容workerman3.x。**

##  获得请求对象
请求对象一律在onMessage回调函数中获取，框架会自动将Request对象通过回调函数第二个参数传递进来。

**例子**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // $request为请求对象，这里没有对请求对象执行任何操作直接返回hello给浏览器
    $connection->send("hello");
};

// 运行worker
Worker::runAll();
```

当浏览器访问`http://127.0.0.1:8080`时将返回`hello`。

## 获得请求参数get

**获取整个get数组**
```php
$get = $request->get();
```
如果请求没有get参数则返回一个空的数组。

**获取get数组的某一个值**
```php
$name = $request->get('name');
```
如果get数组中不包含这个值则返回null。

你也可以给get方法第二个参数传递一个默认值，如果get数组中没找到对应值则返回默认值。例如：
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

// 运行worker
Worker::runAll();
```

当浏览器访问`http://127.0.0.1:8080?name=jerry&age=12`时将返回`jerry`。

## 获得请求参数post
**获取整个post数组**
```php
$post = $request->post();
```
如果请求没有post参数则返回一个空的数组。

**获取post数组的某一个值**
```php
$name = $request->post('name');
```
如果post数组中不包含这个值则返回null。

与get方法一样，你也可以给post方法第二个参数传递一个默认值，如果post数组中没找到对应值则返回默认值。例如：
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

// 运行worker
Worker::runAll();
```


## 获得原始请求post包体
```php
$post = $request->rawBody();
```
这个功能类似与 `php-fpm`里的 `file_get_contents("php://input");`操作。用于获得http原始请求包体。这在获取非`application/x-www-form-urlencoded`格式的post请求数据时很有用。 

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

// 运行worker
Worker::runAll();
```

## 获取header
**获取整个header数组**
```php
$headers = $request->header();
```
如果请求没有header参数则返回一个空的数组。注意所有key均为小写。

**获取header数组的某一个值**
```php
$host = $request->header('host');
```
如果header数组中不包含这个值则返回null。注意所有key均为小写。

与get方法一样，你也可以给header方法第二个参数传递一个默认值，如果header数组中没找到对应值则返回默认值。例如：
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

// 运行worker
Worker::runAll();
```

## 获取cookie
**获取整个cookie数组**
```php
$cookies = $request->cookie();
```
如果请求没有cookie参数则返回一个空的数组。

**获取cookie数组的某一个值**
```php
$name = $request->cookie('name');
```
如果cookie数组中不包含这个值则返回null。

与get方法一样，你也可以给cookie方法第二个参数传递一个默认值，如果cookie数组中没找到对应值则返回默认值。例如：
```php
$name = $request->cookie('name', 'tom');
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
    $cookie = $request->cookie();
    $connection->send(var_export($cookie, true));
};

// 运行worker
Worker::runAll();
```

## 获取上传文件
**获取整个上传文件数组**
```php
$files = $request->file();
```
返回的文件格式类似:
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
其中：
 
 - name为文件名字
 - tmp_name为磁盘临时文件位置
 - size为文件大小
 - error为[错误码](https://www.php.net/manual/zh/features.file-upload.errors.php)
 - type为文件mine类型。

**注意：**

 - 上传文件大小受到[defaultMaxPackageSize](../tcp-connection/default-max-package-size.md)限制，默认10M，可修改。

 - 请求结束后文件将被自动清除。

 - 如果请求没有上传文件则返回一个空的数组。

### 获取特定上传文件
```php
$avatar_file = $request->file('avatar');
```
返回类似
```php
array (
        'name' => '123.jpg',
        'tmp_name' => '/tmp/workerman.upload.9hjR4w',
        'size' => 1196127,
        'error' => 0,
        'type' => 'application/octet-stream',
  )
```
如果上传文件不存在则返回null。

**例子**
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

// 运行worker
Worker::runAll();
```
## 获取host
获取请求的host信息。
```php
$host = $request->host();
```
如果请求的地址是非标准的80或者443端口，host信息可能会携带端口，例如`example.com:8080`。如果不需要端口第一个参数可以传入`true`。

```php
$host = $request->host(true);
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
    $connection->send($request->host());
};

// 运行worker
Worker::runAll();
```
当浏览器访问`http://127.0.0.1:8080?name=tom`时将返回`127.0.0.1:8080`。

## 获取请求方法
```php
$method = $request->method();
```
返回值可能是`GET`、`POST`、`PUT`、`DELETE`、`OPTIONS`、`HEAD`中的一个。

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

// 运行worker
Worker::runAll();
```
## 获取请求uri
```php
$uri = $request->uri();
```
返回请求的uri，包括path和queryString部分。

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

// 运行worker
Worker::runAll();
```
当浏览器访问`http://127.0.0.1:8080/user/get.php?uid=10&type=2`时将返回`/user/get.php?uid=10&type=2`。

## 获取请求路径

```php
$path = $request->path();
```
返回请求的path部分。

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

// 运行worker
Worker::runAll();
```
当浏览器访问`http://127.0.0.1:8080/user/get.php?uid=10&type=2`时将返回`/user/get.php`。

## 获取请求queryString

```php
$query_string = $request->queryString();
```
返回请求的queryString部分。

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

// 运行worker
Worker::runAll();
```
当浏览器访问`http://127.0.0.1:8080/user/get.php?uid=10&type=2`时将返回`uid=10&type=2`。

## 获取请求HTTP版本

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

// 运行worker
Worker::runAll();
```

## 获取请求sessionId

```php
$sid = $request->sessionId();
```
返回字符串，由字母和数字组成

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

// 运行worker
Worker::runAll();
```










