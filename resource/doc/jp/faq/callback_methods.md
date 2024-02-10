# PHPコールバックのいくつかの書き方
PHPでは、無名関数の書き方が最も便利ですが、無名関数方式以外にもPHPには他のコールバックの書き方があります。以下はPHPのいくつかのコールバックの書き方の例です。

## 1、無名関数のコールバック

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// 無名関数のコールバック
$http_worker->onMessage = function(TcpConnection $connection, Request $data)
{
    // ブラウザにhello worldを送信
    $connection->send('hello world');
};

Worker::runAll();
```

## 2、通常関数のコールバック

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// 無名関数のコールバック
$http_worker->onMessage = 'on_message';

// 通常関数
function on_message(TcpConnection $connection, Request $request)
{
    // ブラウザにhello worldを送信
    $connection->send('hello world');
}

Worker::runAll();
```

## 3、コールバックとしてクラスのメソッドを使用する

MyClass.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
class MyClass{
    public function __construct(){}
    public function onWorkerStart(Worker $worker){}
    public function onConnect(TcpConnection $connection){}
    public function onMessage(TcpConnection $connection, $message) {}
    public function onClose(TcpConnection $connection){}
    public function onWorkerStop(Worker $worker){}
}
```

起動スクリプト start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// MyClassを読み込む
require_once __DIR__.'/MyClass.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// オブジェクトを作成
$my_object = new MyClass();

// クラスのメソッドを呼び出す
$worker->onWorkerStart = array($my_object, 'onWorkerStart');
$worker->onConnect     = array($my_object, 'onConnect');
$worker->onMessage     = array($my_object, 'onMessage');
$worker->onClose       = array($my_object, 'onClose');
$worker->onWorkerStop  = array($my_object, 'onWorkerStop');

Worker::runAll();
```

注意：以上のコード構造では、リソース（MySQL接続、Redis接続、Memcache接続など）をコンストラクタで初期化することはできません。なぜなら```$my_object = new MyClass();```はメインプロセスで実行されるからです。 MySQLを例に取ると、メインプロセスで初期化されたMySQL接続などのリソースは子プロセスに継承され、各子プロセスがこのデータベース接続を操作することができますが、サーバ側のMySQLサービスでは同じ接続になるため予期せぬエラーが発生する可能性があります。「mysql gone away」エラーなどです。

上記のコード構造でクラスのコンストラクタ内でリソースを初期化する場合、次のように書くことができます。
MyClass.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
class MyClass{
    protected $db = null;
    public function __construct(){
        // 仮にデータベース接続クラスがMyDbClassであるとします
        $db = new MyDbClass();
    }
    public function onWorkerStart(Worker $worker){}
    public function onConnect(TcpConnection $connection){}
    public function onMessage(TcpConnection $connection, $message) {}
    public function onClose(TcpConnection $connection){}
    public function onWorkerStop(Worker $worker){}
}
```
起動スクリプト start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// クラスをonWorkerStartで初期化
$worker->onWorkerStart = function($worker) {
    // MyClassを読み込む
    require_once __DIR__.'/MyClass.php';
    
    // オブジェクトを作成
    $my_object = new MyClass();

    // クラスのメソッドを呼び出す
    $worker->onConnect    = array($my_object, 'onConnect');
    $worker->onMessage    = array($my_object, 'onMessage');
    $worker->onClose      = array($my_object, 'onClose');
    $worker->onWorkerStop = array($my_object, 'onWorkerStop');
};

Worker::runAll();
```
上記のコード構造では、onWorkerStartがサブプロセスにすでに属しているため、各サブプロセスが独自のMySQL接続を持っていて共有接続の問題が発生しません。
これにより、ビジネスコードのリロードもサポートされます。 MyClass.phpはサブプロセスにロードされるため、リロード規則に従ってMyClass.phpを変更し、直接リロードすることで変更を即座に反映させることができます。

## 4、クラスの静的メソッドをコールバックとして使用する
静的クラスMyClass.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
class MyClass{
    public static function onWorkerStart(Worker $worker){}
    public static function onConnect(TcpConnection $connection){}
    public static function onMessage(TcpConnection $connection, $message) {}
    public static function onClose(TcpConnection $connection){}
    public static function onWorkerStop(Worker $worker){}
}
```
起動スクリプト start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// MyClassを読み込む
require_once __DIR__.'/MyClass.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// クラスの静的メソッドを呼び出す。
$worker->onWorkerStart = array('MyClass', 'onWorkerStart');
$worker->onConnect     = array('MyClass', 'onConnect');
$worker->onMessage     = array('MyClass', 'onMessage');
$worker->onClose       = array('MyClass', 'onClose');
$worker->onWorkerStop  = array('MyClass', 'onWorkerStop');

// クラスに名前空間が含まれている場合は、次のように書きます
// $worker->onWorkerStart = array('your\namesapce\MyClass', 'onWorkerStart');
// $worker->onConnect     = array('your\namesapce\MyClass', 'onConnect');
// $worker->onMessage     = array('your\namesapce\MyClass', 'onMessage');
// $worker->onClose       = array('your\namesapce\MyClass', 'onClose');
// $worker->onWorkerStop  = array('your\namesapce\MyClass', 'onWorkerStop');

Worker::runAll();
```

注意：PHPの動作原則に従い、newがないとコンストラクタは呼び出されません。また、静的クラスのメソッドでは```$this```を使用することはできません。
