# 說明

自從4.x版本開始，workerman加強了對HTTP服務的支持。介紹了請求類別、回應類別、session類別以及[SSE](SSE.md)。如果您想使用workerman的HTTP服務，強烈建議使用workerman4.x或更高版本。

**請注意以下所有用法均適用於workerman 4.x版本，不相容workerman 3.x。**

# 獲取session物件
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
- session 必須在 `$connection->send()` 調用之前操作。
- 當物件銷毀時，session會自動保存更改，因此不要將 `$request->session()` 返回的物件保存在全域陣列或類別成員中，這樣可能導致session無法保存。
- session 默認存儲在磁碟文件中，若要提升性能建議使用redis。


## 獲取所有session數據
```php
$session = $request->session();
$all = $session->all();
```
返回一個陣列。若沒有任何session數據，則返回一個空陣列。

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


## 獲取session中某個值
```php
$session = $request->session();
$name = $session->get('name');
```
若數據不存在則返回null。

您也可以給get方法第二個參數傳遞一個默認值，如果session陣列中找不到對應的值則返回默認值。例如：
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


## 儲存session
儲存某一項數據時使用set方法。
```php
$session = $request->session();
$session->set('name', 'tom');
```
set沒有返回值，session物件銷毀時session會自動保存。

當儲存多個值時使用put方法。
```php
$session = $request->session();
$session->put(['name' => 'tom', 'age' => 12]);
```
同樣地，put也沒有返回值。

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


## 刪除session數據
刪除某個或某些session數據時使用`forget`方法。
```php
$session = $request->session();
// 刪除一項
$session->forget('name');
// 刪除多項
$session->forget(['name', 'age']);
```

此外系統提供了delete方法，與forget方法區別是，delete只能刪除一項。
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


## 獲取並刪除session某個值
```php
$session = $request->session();
$name = $session->pull('name');
```
效果與以下程式碼相同
```php
$session = $request->session();
$value = $session->get($name);
$session->delete($name);
```
若對應session不存在，則返回null。

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


## 刪除所有session數據
```php
$request->session()->flush();
```
沒有返回值，session物件銷毀時session會自動從儲存中刪除。

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


## 判斷對應session數據是否存在
```php
$session = $request->session();
$has = $session->has('name');
```
當該session不存在或對應的session值為null時返回false，否則返回true。

```php
$session = $request->session();
$has = $session->exists('name');
```
以上程式碼也用於判斷session數據是否存在，其區別在於當對應的session項值為null時，也返回true。
