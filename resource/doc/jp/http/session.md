# 説明

Workermanは4.xバージョンからHTTPサービスのサポートが強化されました。リクエストクラス、レスポンスクラス、セッションクラスおよび[SSE](SSE.md)が導入されました。WorkermanのHTTPサービスを使用したい場合は、Workermanの4.xまたはそれ以降のバージョンを強くお勧めします。

**以下はすべてWorkerman 4.xバージョンの使用法であり、Workerman 3.xと互換性がありません。**


# セッションオブジェクトを取得
```php
$session = $request->session();
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
    $session = $request->session();
    $session->set('name', 'tome');
    $connection->send($session->get('name'));
};

// ワーカーを実行
Worker::runAll();
```
**注記**
- セッションは`$connection->send()`呼び出しの前に操作する必要があります。
- オブジェクトが破棄されると、セッションは自動的に変更内容が保存されますので、`$request->session()`で返されたオブジェクトをグローバル配列やクラスメンバーに保存しないようにしてください。
- セッションはデフォルトでディスクファイルに保存されていますが、パフォーマンスを向上させるためにはRedisを使用することをお勧めします。


## すべてのsessionデータを取得
```php
$session = $request->session();
$all = $session->all();
```
返されるのは配列です。何もセッションデータがない場合は空の配列が返されます。

**例**
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

// ワーカーを実行
Worker::runAll();
```


## セッションの特定の値を取得
```php
$session = $request->session();
$name = $session->get('name');
```
データが存在しない場合はnullが返されます。

また、2番目のパラメータにデフォルト値を渡すことができます。セッション配列で対応する値が見つからない場合はデフォルト値が返されます。例：
```php
$session = $request->session();
$name = $session->get('name', 'tom');
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
    $session = $request->session();
    $connection->send($session->get('name', 'tom'));
};

// ワーカーを実行
Worker::runAll();
```


## セッションの保存
特定のデータを保存する場合は、setメソッドを使用します。
```php
$session = $request->session();
$session->set('name', 'tom');
```
セットには返り値がありません。セッションオブジェクトが破棄されると、セッションは自動的に保存されます。

複数の値を保存する場合はputメソッドを使用します。
```php
$session = $request->session();
$session->put(['name' => 'tom', 'age' => 12]);
```
同様に、putにも返り値はありません。

**例**
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

// ワーカーを実行
Worker::runAll();
```


## セッションデータの削除
特定のセッションデータを削除する場合は、`forget`メソッドを使用します。
```php
$session = $request->session();
// 一つ削除
$session->forget('name');
// 複数削除
$session->forget(['name', 'age']);
```
また、システムはforgetメソッドの代わりに一つだけ削除するdeleteメソッドも提供しています。
```php
$session = $request->session();
// $session->forget('name');と同等
$session->delete('name');
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
    $request->session()->forget('name');
    $connection->send('ok');
};

// ワーカーを実行
Worker::runAll();
```


## セッションの特定の値を取得して削除
```php
$session = $request->session();
$name = $session->pull('name');
```
以下のコードと同じ効果があります。
```php
$session = $request->session();
$value = $session->get($name);
$session->delete($name);
```
対応するセッションが存在しない場合、nullが返されます。

**例**
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

// ワーカーを実行
Worker::runAll();
```


## すべてのセッションデータを削除
```php
$request->session()->flush();
```
返り値はありません。セッションオブジェクトが破棄されると、セッションは自動的に保存されます。

**例**
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

// ワーカーを実行
Worker::runAll();
```


## 対応するセッションデータの存在を確認
```php
$session = $request->session();
$has = $session->has('name');
```
対応するセッションが存在しない場合、または対応するセッション値がnullの場合はfalseを返し、それ以外の場合はtrueを返します。

```php
$session = $request->session();
$has = $session->exists('name');
```
上記のコードもセッションデータの存在を確認するために使用されますが、違いは対応するセッション項目の値がnullの場合でもtrueを返すことです。
