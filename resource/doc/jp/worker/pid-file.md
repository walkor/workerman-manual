# pidFile（pidファイル）
## 説明：
```php
static string Worker::$pidFile
```

特別な必要がない限り、このプロパティを設定しないことをお勧めします。

このプロパティはグローバルな静的プロパティであり、WorkerManプロセスのpidファイルのパスを設定するために使用されます。

この設定は監視に便利であり、例えばWorkerManのpidファイルを固定のディレクトリに置くことで、いくつかの監視ソフトウェアがpidファイルを読み取り、それによってWorkerManプロセスの状態を監視することができます。

設定しない場合、WorkerManはデフォルトでWorkermanディレクトリと同じ位置（注意：workerman3.2.3以前のバージョンでは```sys_get_temp_dir()```の下）に自動的にpidファイルを生成します。また、WorkerManインスタンスを複数起動してpidの衝突を避けるために、WorkerManは現在のWorkerManのパスを含めてpidファイルを生成します。

注意：このプロパティは```Worker::runAll();``` を実行する前に設定する必要があります。Windowsシステムではこの機能はサポートされていません。

## 例

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$pidFile = '/var/run/workerman.pid';

$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start";
};
// Workerを実行
Worker::runAll();
```
