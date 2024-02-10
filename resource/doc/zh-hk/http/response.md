# 說明

自4.x版本起，workerman增強了對HTTP服務的支持。引入了請求類、響應類、會話類以及[SSE](SSE.md)。如果您要使用workerman的HTTP服務，強烈建議使用workerman4.x或更新版本。

**請注意以下都是workerman4.x版本的使用方式，不兼容workerman3.x。**

# 注意

- 除非發送的是chunk或者SSE響應，否則在一個請求中多次發送響應是不允許的，也就是在一個請求中不允許多次調用`$connection->send()`。
- 每個請求最終都需要調用一次`$connection->send()`發送響應，否則客戶端會一直等待

## 快捷響應

當不需要更改HTTP狀態碼(默認200)，或者自定義header、cookie時，可以直接向客戶端發送字符串完成響應。

**例子**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // 直接發送this is body給客戶端
    $connection->send("this is body");
};

// 運行worker
Worker::runAll();
```

## 更改狀態碼

當需要自定義狀態碼、header、cookie時，需要使用`Workerman\Protocols\Http\Response`響應類。例如下面例子在訪問路徑為`/404`時返回404的狀態碼，包體內容為`<h1>抱歉，文件不存在</h1>`。

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

// 運行worker
Worker::runAll();
```
當`Response`類已經初始化後，想更改狀態碼使用下面方法。
```php
$response = new Response(200);
$response->withStatus(404);
$connection->send($response);
```

## 發送header

同樣的，發送header需要使用`Workerman\Protocols\Http\Response`響應類。

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

// 運行worker
Worker::runAll();
```
當`Response`類已經初始化後，想增加或者更改header使用下面方法。
```php
$response = new Response(200);
// 添加或者更改一個header
$response->header('Content-Type', 'text/html');
// 添加或者更改多個header
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

## 發送cookie

同樣的，發送cookie需要使用`Workerman\Protocols\Http\Response`響應類。

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

// 運行worker
Worker::runAll();
```

## 發送文件

同樣的，發送文件需要使用`Workerman\Protocols\Http\Response`響應類。

發送文件時用以下方式
```php
$response = (new Response())->withFile($file);
$connection->send($response);
```
- workerman支持發送超大文件
- 對於大文件(超過2M)，workerman不會將整個文件一次性讀入內存，而是在適當的時機分段讀取文件並發送
- workerman會根據客戶端接收速度來優化文件讀取發送速度，保證最快速發送文件的同時將內存佔用減少到最低
- 數據發送是非阻塞的，不會影響其他請求處理
- 發送文件時會自動加上`Last-Modified`頭，以便下次請求時服務端判斷是否發送304響應以節省文件傳輸提高性能
- 發送的文件會自動使用合適的`Content-Type`頭發送給瀏覽器
- 如果文件不存在，會自動轉為404響應

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
    // 檢查if-modified-since頭判斷文件是否修改過
    if (!empty($if_modified_since = $request->header('if-modified-since'))) {
        $modified_time = date('D, d M Y H:i:s',  filemtime($file)) . ' ' . \date_default_timezone_get();
        // 文件未修改則返回304
        if ($modified_time === $if_modified_since) {
            $connection->send(new Response(304));
            return;
        }
    }
    // 文件修改過或者沒有if-modified-since頭則發送文件
    $response = (new Response())->withFile($file);
    $connection->send($response);
};

// 運行worker
Worker::runAll();
```

## 發送http chunk數據
- 必須先發送一個攜帶 `Transfer-Encoding: chunked`頭的Response響應給客戶端
- 發送後續chunk數據使用`Workerman\Protocols\Http\Chunk` 類
- 最終必須發送一個空的chunk來結束響應

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
    // 首先發送一個帶Transfer-Encoding: chunked頭的Response響應
    $connection->send(new Response(200, array('Transfer-Encoding' => 'chunked'), 'hello'));
    // 後續Chunk數據用Workerman\Protocols\Http\Chunk類發送
    $connection->send(new Chunk('第一段數據'));
    $connection->send(new Chunk('第二段數據'));
    $connection->send(new Chunk('第三段數據'));
    // 最後必須發送一個空的chunk結束響應
    $connection->send(new Chunk(''));
};

// 運行worker
Worker::runAll();
```
