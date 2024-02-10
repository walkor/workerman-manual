# 説明
workermanは4.xバージョンからHTTPサービスのサポートを強化しました。 リクエストクラス、レスポンスクラス、セッションクラス、[SSE](SSE.md)などが導入されました。WorkermanのHTTPサービスを使用したい場合は、Workerman 4.x以降のバージョンを強く推奨します。

**以下は全てWorkerman 4.xバージョンの使用法であり、Workerman 3.xとの互換性はありません。**

## セッションストレージの変更
workermanはセッションのためにファイルストレージエンジンとRedisストレージエンジンを提供しています。デフォルトではファイルストレージエンジンが使用されます。Redisストレージエンジンに変更したい場合は、以下のコードを参照してください。
```php
<?php
use Workerman\Worker;
use Workerman\Protocols\Http\Session;
use Workerman\Protocols\Http\Session\RedisSessionHandler;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

// redisの設定
$config = [
    'host'     => '127.0.0.1', // 必須パラメータ
    'port'     => 6379,        // 必須パラメータ
    'timeout'  => 2,           // オプションパラメータ
    'auth'     => '******',    // オプションパラメータ
    'database' => 1,           // オプションパラメータ
    'prefix'   => 'session_'   // オプションパラメータ
];
// Workerman\Protocols\Http\Session::handlerClassメソッドを使用してセッションの内部ドライバークラスを変更する
Session::handlerClass(RedisSessionHandler::class, $config);

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('somekey', rand());
    $connection->send($session->get('somekey'));
};

Worker::runAll();
```


## セッションストレージの位置を設定する
デフォルトのストレージエンジンを使用する場合、セッションデータはディスク上にデフォルトで`session_save_path()`の返り値として保存されます。
以下の方法を使用して保存位置を変更できます。
```php
use Workerman\Worker;
use \Workerman\Protocols\Http\Session\FileSessionHandler;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

// セッションファイルの保存場所を設定する
FileSessionHandler::sessionSavePath('/tmp/session');

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('name', 'tome');
    $connection->send($session->get('name'));
};

// workerを実行する
Worker::runAll();
```


## セッションファイルのクリーンアップ
デフォルトのセッションストレージエンジンを使用すると、ディスク上に複数のセッションファイルが存在し、workermanはphp.iniで設定された`session.gc_probability` `session.gc_divisor` `session.gc_maxlifetime`オプションに基づいて期限切れのセッションファイルをクリーンアップします。これらのオプションの詳細については、[PHPマニュアル](https://www.php.net/manual/zh/session.configuration.php#ini.session.gc-probability)を参照してください。


## ストレージドライバーの変更
ファイルセッションストレージエンジンとRedisセッションストレージエンジン以外にも、workermanでは標準の[SessionHandlerInterface](https://www.php.net/manual/zh/class.sessionhandlerinterface.php)インターフェースを使用して新しいセッションストレージエンジンを追加することができます。例えば、mangoDbセッションストレージエンジンや、MySQLセッションストレージエンジンなどです。

**新しいセッションストレージエンジンを追加する手順**
 1. [SessionHandlerInterface](https://www.php.net/manual/zh/class.sessionhandlerinterface.php) インターフェースを実装する
 2. `Workerman\Protocols\Http\Session::handlerClass($class_name, $config)` メソッドを使用して底層のSessionHandlerインターフェースを置き換える

**SessionHandlerInterfaceの実装**

カスタムセッションストレージドライバーはSessionHandlerInterfaceインターフェースを実装する必要があります。このインターフェースには次のメソッドが含まれています。
```php
SessionHandlerInterface {
    abstract public read ( string $session_id ) : string
    abstract public write ( string $session_id , string $session_data ) : bool
    abstract public destroy ( string $session_id ) : bool
    abstract public gc ( int $maxlifetime ) : int
    abstract public close ( void ) : bool
    abstract public open ( string $save_path , string $session_name ) : bool
}
```
**SessionHandlerInterfaceの説明**
 - readメソッドはストレージからsession_idに対応するセッションデータを読み込むためのものです。データの逆シリアル化を行わないでください、フレームワークが自動的に処理します。
 - writeメソッドはsession_idに対応するセッションデータをストレージに書き込むためのものです。データのシリアル化を行わないでください、フレームワークが自動的に処理します。
 - destroyメソッドはsession_idに対応するセッションデータを破壊するためのものです。
 - gcメソッドは期限切れのセッションデータを削除するためのものです。ストレージはmaxlifetimeよりも最終更新時間が大きいすべてのセッションに対して削除操作を実行する必要があります。
 - closeメソッドは何も行う必要がなく、単にtrueを返します。
 - openメソッドは何も行う必要がなく、単にtrueを返します。

**底層ドライバーの置き換え**

SessionHandlerInterfaceインターフェースを実装した後、以下の方法を使用してセッションの底層ドライバーを置き換えます。

```php
Workerman\Protocols\Http\Session::handlerClass($class_name, $config);
```
 - $class_name はSessionHandlerInterfaceインターフェースを実装したSessionHandlerクラスの名前です。名前空間がある場合は完全な名前空間を使用する必要があります。
 - $config はSessionHandlerクラスのコンストラクタのパラメータです

**具体的な実装**

*注意: このMySessionHandlerクラスは、セッションの底層ドライバーを変更する手順を説明するためのものであり、実稼働環境で使用することはできません。*
```php
<?php
use Workerman\Worker;
use Workerman\Protocols\Http\Session;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

class MySessionHandler implements SessionHandlerInterface
{
    protected static $store = [];
    
    public function __construct($config) {
        // ['host' => 'localhost']
        var_dump($config);
    }
   
    public function open($save_path, $name)
    {
        return true;
    }

    public function read($session_id)
    {
        return isset(static::$store[$session_id]) ? static::$store[$session_id]['content'] : '';
    }

    public function write($session_id, $session_data)
    {
        static::$store[$session_id] = ['content' => $session_data, 'timestamp' => time()];
    }

    public function close()
    {
        return true;
    }

    public function destroy($session_id)
    {
        unset(static::$store[$session_id]);
        return true;
    }

    public function gc($maxlifetime) {
        $time_now = time();
        foreach (static::$store as $session_id => $info) {
            if ($time_now - $info['timestamp'] > $maxlifetime) {
                unset(static::$store[$session_id]);
            }
        }
    }
}

// 新しいSessionHandlerクラスにはいくつかの構成が必要と想定されているとします
$config = ['host' => 'localhost'];
// Workerman\Protocols\Http\Session::handlerClass($class_name, $config) メソッドを使用してセッションの底層ドライバークラスを変更する
Session::handlerClass(MySessionHandler::class, $config);

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('somekey', rand());
    $connection->send($session->get('somekey'));
};

Worker::runAll();
```
