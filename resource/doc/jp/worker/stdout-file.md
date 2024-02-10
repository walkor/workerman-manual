# stdoutFile
## 説明:
```php
static string Worker::$stdoutFile
```

このプロパティはグローバルな静的プロパティであり、デーモンモード(```-d```で起動)で実行されている場合、すべてのターミナル出力（echo、var_dumpなど）はstdoutFileで指定されたファイルにリダイレクトされます。

設定しない場合、かつデーモンモードで実行している場合、すべてのターミナル出力は`/dev/null`にリダイレクトされます（つまり、デフォルトではすべての出力が破棄されます）。

> 注意：`/dev/null`はLinuxで特別なファイルであり、実際にはブラックホールです。このファイルに書き込まれたすべてのデータは破棄されます。出力を破棄したくない場合は、```Worker::$stdoutFile```を正常なファイルパスに設定できます。

> 注意：このプロパティは```Worker::runAll();```が実行される前に設定する必要があります。Windowsシステムではこの機能はサポートされていません。

## 例

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$daemonize = true;
// すべての出力を/tmp/stdout.logファイルに保存
Worker::$stdoutFile = '/tmp/stdout.log';
$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start\n";
};
// Workerを実行
Worker::runAll();
```
