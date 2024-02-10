# 説明
Workermanは4.xバージョンからHTTPサービスのサポートを強化しました。リクエストクラス、レスポンスクラス、セッションクラス、[SSE](SSE.md) を導入しました。WorkermanのHTTPサービスを使用する場合、Workerman 4.x以降のバージョンを強く推奨します。

**以下はすべてWorkerman 4.xバージョンの使用方法であり、Workerman 3.xと互換性がありません。**

## リクエストオブジェクトの取得
リクエストオブジェクトは、onMessageコールバック関数内で取得し、フレームワークは自動的にRequestオブジェクトをコールバック関数の2番目の引数として渡します。

**例**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // $requestはリクエストオブジェクトであり、ここではリクエストオブジェクトに対して操作を行わずに直接"hello"をブラウザに送信します
    $connection->send("hello");
};

// Workerを実行
Worker::runAll();
```

ブラウザが`http://127.0.0.1:8080`にアクセスすると、`hello`が返されます。

## GETパラメーターの取得
**GET配列全体を取得**
```php
$get = $request->get();
```
リクエストにGETパラメーターがない場合は、空の配列が返されます。

**GET配列の特定の値を取得**
```php
$name = $request->get('name');
```
GET配列にその値が含まれていない場合は、nullが返されます。

同様に、getメソッドに2番目の引数としてデフォルト値を渡すこともできます。GET配列に対応する値が見つからない場合は、デフォルト値が返されます。例：
```php
$name = $request->get('name', 'tom');
```

**例**
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

// Workerを実行
Worker::runAll();
```

ブラウザが`http://127.0.0.1:8080?name=jerry&age=12`にアクセスすると、`jerry`が返されます。

## POSTパラメーターの取得
**POST配列全体を取得**
```php
$post = $request->post();
```
リクエストにPOSTパラメーターがない場合は、空の配列が返されます。

**POST配列の特定の値を取得**
```php
$name = $request->post('name');
```
POST配列にその値が含まれていない場合は、nullが返されます。

GETメソッドと同様に、postメソッドに2番目の引数としてデフォルト値を渡すこともできます。POST配列に対応する値が見つからない場合は、デフォルト値が返されます。例：
```php
$name = $request->post('name', 'tom');
```

**例**
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

// Workerを実行
Worker::runAll();
```

## リクエストの生のPOSTボディを取得
```php
$post = $request->rawBody();
```
この機能は、`php-fpm`の`file_get_contents("php://input");`の動作に類似しており、HTTPの生のリクエストボディを取得するために使用されます。これは、`application/x-www-form-urlencoded`形式以外のPOSTリクエストデータを取得する際に役立ちます。

**例**
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

// Workerを実行
Worker::runAll();
```

## ヘッダーの取得
**ヘッダー配列全体を取得**
```php
$headers = $request->header();
```
リクエストにヘッダーパラメーターがない場合は、空の配列が返されます。すべてのキーは小文字であることに注意してください。

**ヘッダー配列の特定の値を取得**
```php
$host = $request->header('host');
```
ヘッダー配列にその値が含まれていない場合は、nullが返されます。すべてのキーは小文字であることに注意してください。

GETメソッドと同様に、headerメソッドに2番目の引数としてデフォルト値を渡すこともできます。ヘッダー配列に対応する値が見つからない場合は、デフォルト値が返されます。例：
```php
$host = $request->header('host', 'localhost');
```

**例**
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

// Workerを実行
Worker::runAll();
```

## クッキーの取得
**クッキー配列全体を取得**
```php
$cookies = $request->cookie();
```
リクエストにクッキーパラメーターがない場合は、空の配列が返されます。

**クッキー配列の特定の値を取得**
```php
$name = $request->cookie('name');
```
クッキー配列にその値が含まれていない場合は、nullが返されます。

GETメソッドと同様に、cookieメソッドに2番目の引数としてデフォルト値を渡すこともできます。クッキー配列に対応する値が見つからない場合は、デフォルト値が返されます。例：
```php
$name = $request->cookie('name', 'tom');
```

**例**
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

// Workerを実行
Worker::runAll();
```

## アップロードファイルの取得
**アップロードファイル配列全体を取得**
```php
$files = $request->file();
```
ファイルの形式は以下のようになります:
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
ここで:

 - nameはファイル名です
 - tmp_nameはディスク上の一時ファイルの位置です
 - sizeはファイルのサイズです
 - errorは[エラーコード](https://www.php.net/manual/zh/features.file-upload.errors.php)です
 - typeはファイルのMIMEタイプです。

**注意:**

 - アップロードファイルのサイズは[defaultMaxPackageSize](../tcp-connection/default-max-package-size.md)に制限され、デフォルトは10Mですが、変更可能です。

 - リクエストの終了後、ファイルは自動的に削除されます。

 - リクエストにアップロードファイルがない場合は、空の配列が返されます。

### 特定のアップロードファイルを取得
```php
$avatar_file = $request->file('avatar');
```
以下のように返されます
```php
array (
        'name' => '123.jpg',
        'tmp_name' => '/tmp/workerman.upload.9hjR4w',
        'size' => 1196127,
        'error' => 0,
        'type' => 'application/octet-stream',
  )
```
アップロードされたファイルが存在しない場合はnullが返されます。

**例**
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

// Workerを実行
Worker::runAll();
```
## ホストの取得
リクエストのホスト情報を取得します。
```php
$host = $request->host();
```
リクエストが標準でない80または443ポートである場合、ホスト情報にポートが含まれることがあります。例えば、 `example.com:8080` のようになります。ポートを除外したい場合は、最初のパラメータに `true` を渡すことができます。
```php
$host = $request->host(true);
```
**例**
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

// ワーカーを実行
Worker::runAll();
```
ウェブブラウザが`http://127.0.0.1:8080?name=tom` にアクセスした場合、`127.0.0.1:8080` が返されます。

## リクエストメソッドの取得
```php
$method = $request->method();
```
戻り値は `GET`、`POST`、`PUT`、`DELETE`、`OPTIONS`、`HEAD` のいずれかです。

**例**
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

// ワーカーを実行
Worker::runAll();
```

## リクエストURIの取得
```php
$uri = $request->uri();
```
リクエストのURIを、パスとクエリ文字列を含めて返します。

**例**
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

// ワーカーを実行
Worker::runAll();
```
ウェブブラウザが `http://127.0.0.1:8080/user/get.php?uid=10&type=2` にアクセスした場合、 `/user/get.php?uid=10&type=2` が返されます。

## リクエストパスの取得
```php
$path = $request->path();
```
リクエストのパス部分を返します。

**例**
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

// ワーカーを実行
Worker::runAll();
```
ウェブブラウザが `http://127.0.0.1:8080/user/get.php?uid=10&type=2` にアクセスした場合、 `/user/get.php` が返されます。

## リクエストクエリ文字列の取得
```php
$query_string = $request->queryString();
```
リクエストのクエリ文字列部分を返します。

**例**
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

// ワーカーを実行
Worker::runAll();
```
ウェブブラウザが `http://127.0.0.1:8080/user/get.php?uid=10&type=2` にアクセスした場合、`uid=10&type=2` が返されます。

## リクエストのHTTPバージョンの取得
```php
$version = $request->protocolVersion();
```
文字列 `1.1` または `1.0` を返します。

**例**
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

// ワーカーを実行
Worker::runAll();
```

## リクエストのセッションIDの取得
```php
$sid = $request->sessionId();
```
文字列、英数字で構成されています。

**例**
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

// ワーカーを実行
Worker::runAll();
```
