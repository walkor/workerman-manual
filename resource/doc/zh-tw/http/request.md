# 說明
從4.x版本開始，workerman加強了對HTTP服務的支持。引入了請求類、響應類、session類以及[SSE](SSE.md)。如果你想使用workerman的HTTP服務，強烈推薦使用workerman4.x或更高版本。 

**請注意以下皆為workerman4.x版本的用法，不相容workerman3.x。**

## 獲得請求對象
請求對象一律在onMessage回調函數中獲得，框架會自動將Request對象通過回調函數第二個參數傳遞過來。

**範例**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
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

你也可以給get方法第二個參數傳遞一個默認值，如果get陣列中沒有找到對應值則返回默認值。例如：
```php
$name = $request->get('name', 'tom');
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
    $connection->send($request->get('name'));
};

// 運行worker
Worker::runAll();
```

當瀏覽器訪問`http://127.0.0.1:8080?name=jerry&age=12`時將返回`jerry`。

## 獲得請求參數post
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

與get方法一樣，你也可以給post方法第二個參數傳遞一個默認值，如果post陣列中沒有找到對應值則返回默認值。例如：
```php
$name = $request->post('name', 'tom');
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
    $post = $request->post();
    $connection->send(var_export($post, true));
};

// 運行worker
Worker::runAll();
```


## 獲得原始請求post包體
```php
$post = $request->rawBody();
```
這個功能類似於 `php-fpm`裡的 `file_get_contents("php://input");`操作。用於獲得http原始請求包體。這在獲取非`application/x-www-form-urlencoded`格式的post請求數據時很有用。

**範例**
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

與get方法一樣，你也可以給header方法第二個參數傳遞一個默認值，如果header陣列中沒有找到對應值則返回默認值。例如：
```php
$host = $request->header('host', 'localhost');
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
    if ($request->header('connection') === 'keep-alive') {
        $connection->send('hello');
    } else {
        $connection->close('hello');
    }    
};

// 運行worker
Worker::runAll();
``` 

## 獲取cookie
**獲取整個cookie陣列**
```php
$cookies = $request->cookie();
```
如果請求沒有cookie參數則返回一個空的陣列。

**獲取cookie陣列的某一個值**
```php
$name = $request->cookie('name');
```
如果cookie陣列中不包含這個值則返回null。

與get方法一樣，你也可以給cookie方法第二個參數傳遞一個默認值，如果cookie陣列中沒有找到對應值則返回默認值。例如：
```php
$name = $request->cookie('name', 'tom');
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
    $cookie = $request->cookie();
    $connection->send(var_export($cookie, true));
};

// 運行worker
Worker::runAll();
``` 

## 獲取上傳文件
**獲取整個上傳文件陣列**
```php
$files = $request->file();
```
返回的文件格式類似:
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

 - name為文件名稱
 - tmp_name為磁碟臨時文件位置
 - size為文件大小
 - error為[錯誤碼](https://www.php.net/manual/zh/features.file-upload.errors.php)
 - type為文件mine類型。

**注意：**

 - 上傳文件大小受到[defaultMaxPackageSize](../tcp-connection/default-max-package-size.md)限制，默认10M，可修改。

 - 請求結束後文件將被自動清除。

 - 如果請求沒有上傳文件則返回一個空的陣列。

### 獲取特定上傳文件
```php
$avatar_file = $request->file('avatar');
```
返回類似
```php
array (
        'name' => '123.jpg',
        'tmp_name' => '/tmp/workerman.upload.9hjR4w',
        'size' => 1196127,
        'error' => 0,
        'type' => 'application/octet-stream',
  )
```
如果上傳文件不存在則返回null。

**範例**
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

// 運行worker
Worker::runAll();
``` 

## 獲取host
獲取請求的host信息。
```php
$host = $request->host();
```
如果請求的地址是非標準的80或443端口，host信息可能會攜帶端口，例如`example.com:8080`。如果不需要端口第一個參數可以傳入`true`。

```php
$host = $request->host(true);
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
    $connection->send($request->host());
};

// 運行worker
Worker::runAll();
``` 

當瀏覽器訪問`http://127.0.0.1:8080?name=tom`時將返回`127.0.0.1:8080`。
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
