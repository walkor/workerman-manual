# 說明

自Workerman 4.x版本開始，加強了對HTTP服務的支援。引入了請求類、響應類、會話類以及[SSE](SSE.md)。如果你想使用Workerman的HTTP服務，強烈建議使用Workerman4.x或以上版本。

**請注意以下都是Workerman4.x版本的用法，不相容Workerman3.x。**

# 獲取Session對象
```php
$session = $request->session();
```

**範例**
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

// 運行worker
Worker::runAll();
```
**注意事項**
- Session必須在`$connection->send()`調用之前操作。
- Session在物件被銷毀時會自動保存修改，所以不要把`$request->session()`返回的物件保存在全局陣列或者類成員中導致Session無法保存。
- Session默認存儲在磁盤文件中，如果想要更好的性能建議使用redis。

## 獲取所有Session數據
```php
$session = $request->session();
$all = $session->all();
```
返回的是一個陣列。如果沒有任何Session數據，則返回一個空陣列。

**範例**
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

// 運行worker
Worker::runAll();
```

## 獲取Session中某個值
```php
$session = $request->session();
$name = $session->get('name');
```
如果數據不存在則返回null。

你也可以給get方法第二個參數傳遞一個默認值，如果Session陣列中沒找到對應值則返回默認值。例如：
```php
$session = $request->session();
$name = $session->get('name', 'tom');
```
**範例**
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

// 運行worker
Worker::runAll();
```

## 儲存Session數據
儲存某一項數據時用set方法。
```php
$session = $request->session();
$session->set('name', 'tom');
```
set沒有返回值，Session物件銷毀時Session會自動保存。

當儲存多個值時使用put方法。
```php
$session = $request->session();
$session->put(['name' => 'tom', 'age' => 12]);
```
同樣的，put也沒有返回值。

**範例**
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

// 運行worker
Worker::runAll();
```

## 刪除Session數據
刪除某個或者某些Session數據時用`forget`方法。
```php
$session = $request->session();
// 刪除一項
$session->forget('name');
// 刪除多項
$session->forget(['name', 'age']);
```

另外系統提供了delete方法，與forget方法區別是，delete只能刪除一項。
```php
$session = $request->session();
// 等同於 $session->forget('name');
$session->delete('name');
```

**範例**
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

// 運行worker
Worker::runAll();
```

## 獲取並刪除Session某個值
```php
$session = $request->session();
$name = $session->pull('name');
```
效果與如下代碼相同
```php
$session = $request->session();
$value = $session->get($name);
$session->delete($name);
```
如果對應Session不存在，則返回null。

**範例**
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

// 運行worker
Worker::runAll();
```

## 刪除所有Session數據
```php
$request->session()->flush();
```
沒有返回值，Session物件銷毀時Session會自動從存儲中刪除。

**範例**
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

// 運行worker
Worker::runAll();
```

## 判斷對應Session數據是否存在
```php
$session = $request->session();
$has = $session->has('name');
```
以上當對應的Session不存在或者對應的Session值為null時返回false，否則返回true。

```php
$session = $request->session();
$has = $session->exists('name');
```
以上代碼也是用來判斷Session數據是否存在，區別是當對應的Session項值為null時，也返回true。
