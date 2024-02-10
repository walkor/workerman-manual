# GlobalData コンポーネントクライアント
**```（Workermanのバージョン>=3.3.0が必要です）```**

# __construct
```php
void \GlobalData\Client::__construct(mixed $server_address)
```

\GlobalData\Clientクライアントオブジェクトをインスタンス化します。クライアントオブジェクトにプロパティを割り当てることで、プロセス間でデータを共有できます。

### パラメーター
GlobalDataサーバーのアドレスは、```<IPアドレス>:<ポート>```の形式で指定します。例えば、```127.0.0.1:2207```です。

GlobalDataサーバーがクラスター化されている場合は、アドレスの配列を渡します。例えば、```array('10.0.0.10:2207', '10.0.0.0.11:2207')```です。

## 説明
代入、読み取り、isset、unsetの操作をサポートします。
また、cas原子操作もサポートしています。

## 例

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// GlobalData Server
$global_worker = new GlobalData\Server('0.0.0.0', 2207);

$worker = new Worker('tcp://0.0.0.0:6636');
// ワーカー起動時
$worker->onWorkerStart = function()
{
    // グローバルなglobal data clientを初期化します
    global $global;
    $global = new \GlobalData\Client('127.0.0.1:2207');
};
// サーバーがメッセージを受信するたび
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // $global->somedataの値を変更すると、他のプロセスもこの$global->somedataの変数を共有します
    global $global;
    echo "now global->somedata=".var_export($global->somedata, true)."\n";
    echo "set \$global->somedata=$data";
    $global->somedata = $data;
};
Worker::runAll();
```

### すべての使用法（PHP-FPM環境でも利用可能）
```php
require_once __DIR__ . '/vendor/autoload.php';

$global = new Client('127.0.0.1:2207');

var_export(isset($global->abc));

$global->abc = array(1,2,3);

var_export($global->abc);

unset($global->abc);

var_export($global->add('abc', 10));

var_export($global->increment('abc', 2));

var_export($global->cas('abc', 12, 18));
```

## 注意：
GlobalDataコンポーネントでは、リソースタイプのデータ（たとえば、MySQL接続、ソケット接続など）を共有できません。

Workerman環境でGlobalData/Clientを使用する場合は、onXXXコールバック内でGlobalData/Clientオブジェクトをインスタンス化してください。たとえば、onWorkerStart内でインスタンス化してください。

次のように共有変数を操作することはできません。
```php
$global->somekey = array();
$global->somekey[]='xxx';

$global->someObject = new someClass();
$global->someObject->someVar = 'xxx';
```
次のように操作できます。
```php
$somekey = array();
$somekey[] = 'xxx';
$global->somekey = $somekey;

$someObject = new someClass();
$someObject->someVar = 'xxx';
$global->someObject = $someObject;
```
