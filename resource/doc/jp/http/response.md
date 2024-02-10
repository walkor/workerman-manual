# 説明

workermanは4.xバージョンからHTTPサービスのサポートを強化しました。リクエストクラス、レスポンスクラス、セッションクラスおよび[SSE](SSE.md)を導入しました。workermanのHTTPサービスを使用する場合は、workerman4.xまたはそれ以降のバージョンを強くお勧めします。

**以下はすべてworkerman4.xバージョンの使用法であり、workerman3.xとは互換性がありません。**

# 注意

- チャンクまたはSSEレスポンスを送信しない限り、1つのリクエストで複数回レスポンスを送信することは許可されません。つまり、1つのリクエストで`$connection->send()`を複数回呼び出すことは許可されません。
- 各リクエストは最終的に`$connection->send()`を1回呼び出す必要があります。そうしないと、クライアントはずっと待ち続けます。

## クイックレスポンス
HTTPステータスコードを変更する必要がない（デフォルトは200）、またはカスタムヘッダーやクッキーを設定する必要がない場合は、文字列をクライアントに直接送信してレスポンスを完了させることができます。

**例**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // クライアントに直接 "this is body" を送信
    $connection->send("this is body");
};

// Workerを実行
Worker::runAll();
```

## ステータスコードの変更
カスタムステータスコード、ヘッダー、クッキーを設定する必要がある場合は、`Workerman\Protocols\Http\Response`レスポンスクラスを使用する必要があります。以下の例では、`/404`パスにアクセスしたときに、404のステータスコードを返し、ボディの内容は`<h1>申し訳ございませんが、ファイルが見つかりません</h1>`を返します。

**例**
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
        $connection->send(new Response(404, [], '<h1>申し訳ございませんが、ファイルが見つかりません</h1>'));
    } else {
        $connection->send('this is body');
    }
};

// Workerを実行
Worker::runAll();
```
`Response`クラスが初期化された後にステータスコードを変更したい場合は、以下の方法を使用します。
```php
$response = new Response(200);
$response->withStatus(404);
$connection->send($response);
```

## ヘッダーの送信
同様に、ヘッダーを送信するには`Workerman\Protocols\Http\Response`レスポンスクラスを使用する必要があります。

**例**
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

// Workerを実行
Worker::runAll();
```
`Response`クラスが初期化された後にヘッダーを追加または変更したい場合は、以下の方法を使用します。
```php
$response = new Response(200);
// ヘッダーを追加または変更
$response->header('Content-Type', 'text/html');
// 複数のヘッダーを追加または変更
$response->withHeaders([
    'Content-Type' => 'application/ json',
    'X-Header-One' => 'Header Value'
]);
$connection->send($response);
```

## リダイレクト
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $location = '/test_location';
    $response = new Response(302, ['Location' => $location]);
    $connection->send($response);
};
Worker::runAll();
```

## クッキーの送信
同様に、クッキーを送信するには`Workerman\Protocols\Http\Response`レスポンスクラスを使用する必要があります。

**例**
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

// Workerを実行
Worker::runAll();
```

## ファイルの送信
同様に、ファイルを送信するには`Workerman\Protocols\Http\Response`レスポンスクラスを使用する必要があります。

ファイルを送信する場合は以下の方法を使用します
```php
$response = (new Response())->withFile($file);
$connection->send($response);
```
- workermanは超大きなファイルを送信することができます
- 大きなファイル（2Mを超える）の場合、workermanはファイル全体を一度に読み込むのではなく、適切なタイミングでファイルを分割して送信します
- workermanはクライアントの受信速度に応じてファイルの読み取りと送信速度を最適化し、ファイルを最速で送信するためにメモリ使用量を最小限に保ちます
- データ送信は非ブロッキングであり、他のリクエスト処理に影響を与えません
- ファイルを送信する際、次回のリクエストで304応答を送信してファイル転送を節約し、パフォーマンスを向上させるための`Last-Modified`ヘッダーが自動的に追加されます
- 送信されるファイルは適切な`Content-Type`ヘッダーを使用してブラウザに送信されます
- ファイルが存在しない場合、自動的に404レスポンスに変換されます

**例**
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
    // if-modified-sinceヘッダーをチェックしてファイルが変更されているかどうかを判断する
    if (!empty($if_modified_since = $request->header('if-modified-since'))) {
        $modified_time = date('D, d M Y H:i:s',  filemtime($file)) . ' ' . \date_default_timezone_get();
        // ファイルが変更されていない場合、304を返す
        if ($modified_time === $if_modified_since) {
            $connection->send(new Response(304));
            return;
        }
    }
    // ファイルが変更されたか、if-modified-sinceヘッダーがない場合はファイルを送信する
    $response = (new Response())->withFile($file);
    $connection->send($response);
};

// Workerを実行
Worker::runAll();
```


## http chunkデータの送信
- まず、`Transfer-Encoding: chunked` ヘッダーを持つResponseレスポンスをクライアントに送信する必要があります
- その後のchunkデータの送信には `Workerman\Protocols\Http\Chunk` クラスを使用します
- 最終的には空のchunkを送信してレスポンスを終了する必要があります

**例**
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
    // まず、`Transfer-Encoding: chunked` ヘッダーを持つResponseレスポンスを送信する
    $connection->send(new Response(200, array('Transfer-Encoding' => 'chunked'), 'hello'));
    // その後のChunkデータはWorkerman\Protocols\Http\Chunkクラスで送信する
    $connection->send(new Chunk('第一段データ'));
    $connection->send(new Chunk('第二段データ'));
    $connection->send(new Chunk('第三段データ'));
   // 最後に空のchunkを送信してレスポンスを終了する
    $connection->send(new Chunk(''));
};

// Workerを実行
Worker::runAll();
```
