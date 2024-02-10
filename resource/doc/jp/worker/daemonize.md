# デーモン化
## 説明:
 ```php
static bool Worker::$daemonize
```

この属性はグローバルな静的属性で、デーモン（常駐プロセス）モードで実行するかどうかを示します。起動コマンドに `-d` パラメーターが使用された場合、この属性は自動的に true に設定されます。また、コード内で手動で設定することもできます。

注意: この属性は `Worker::runAll();` の実行前に設定する必要があります。Windowsシステムではこの機能はサポートされていません。

## 例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$daemonize = true;
$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start\n";
};
// ワーカーを実行
Worker::runAll();
```
