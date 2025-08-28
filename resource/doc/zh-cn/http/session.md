# 说明

workerman从4.x版本开始加强了HTTP服务的支持。引入了请求类、响应类、session类以及[SSE](SSE.md)。如果你想使用workerman的HTTP服务，强烈推荐使用workerman4.x或者以后的更高版本。


# 获取session对象
```php
$session = $request->session();
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
    $session = $request->session();
    $session->set('name', 'tome');
    $connection->send($session->get('name'));
};

// 运行worker
Worker::runAll();
```
**注意事项**
- session必须在`$connection->send()`调用之前操作。
- session在对象销毁时会自动保存修改，所以不要把`$request->session()`返回的对象保存在全局数组或者类成员中导致session无法保存。
- session默认存储在磁盘文件中，如果想要更好的性能建议使用redis。


## 获取所有session数据
```php
$session = $request->session();
$all = $session->all();
```
返回的是一个数组。如果没有任何session数据，则返回一个空数组。

**例子**
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

// 运行worker
Worker::runAll();
```


## 获取session中某个值
```php
$session = $request->session();
$name = $session->get('name');
```
如果数据不存在则返回null。

你也可以给get方法第二个参数传递一个默认值，如果session数组中没找到对应值则返回默认值。例如：
```php
$session = $request->session();
$name = $session->get('name', 'tom');
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
    $session = $request->session();
    $connection->send($session->get('name', 'tom'));
};

// 运行worker
Worker::runAll();
```

## 存储session
存储某一项数据时用set方法。
```php
$session = $request->session();
$session->set('name', 'tom');
```
set没有返回值，session对象销毁时session会自动保存。

当存储多个值时使用put方法。
```php
$session = $request->session();
$session->put(['name' => 'tom', 'age' => 12]);
```
同样的，put也没有返回值。

**例子**
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

// 运行worker
Worker::runAll();
```

## 删除session数据
删除某个或者某些session数据时用`forget`方法。
```php
$session = $request->session();
// 删除一项
$session->forget('name');
// 删除多项
$session->forget(['name', 'age']);
```

另外系统提供了delete方法，与forget方法区别是，delete只能删除一项。
```php
$session = $request->session();
// 等同于 $session->forget('name');
$session->delete('name');
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
    $request->session()->forget('name');
    $connection->send('ok');
};

// 运行worker
Worker::runAll();
```

## 获取并删除session某个值
```php
$session = $request->session();
$name = $session->pull('name');
```
效果与如下代码相同
```php
$session = $request->session();
$value = $session->get($name);
$session->delete($name);
```
如果对应session不存在，则返回null。

**例子**
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

// 运行worker
Worker::runAll();
```

## 删除所有session数据
```php
$request->session()->flush();
```
没有返回值，session对象销毁时session会自动从存储中删除。

**例子**
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

// 运行worker
Worker::runAll();
```

## 判断对应session数据是否存在
```php
$session = $request->session();
$has = $session->has('name');
```
以上当对应的session不存在或者对应的session值为null时返回false，否则返回true。

```
$session = $request->session();
$has = $session->exists('name');
```
以上代码也是用来判断session数据是否存在，区别时当对应的session项值为null时，也返回true。

## 注意事项
在使用 session 时不建议直接存储类的实例对象，尤其是来源不可控的类实例，反序列化时可能造成潜在风险。

