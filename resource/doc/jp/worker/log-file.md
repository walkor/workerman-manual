# logFile
## 説明:
```php
static string Worker::$logFile
```

Workermanのログファイルの場所を指定するために使用されます。

このファイルには、Workerman自体に関連するログが記録されます。これには、起動、停止などが含まれます。

設定されていない場合、ファイル名はデフォルトでworkerman.logとなり、ファイルの位置はWorkermanの親ディレクトリにあります。

**注意:**

このログファイルには、Workerman自体に関連する起動や停止などのログのみが含まれており、ビジネスログは含まれません。

ビジネスログは、[file_put_contents](https://php.net/manual/zh/function.file-put-contents.php) や [error_log](https://php.net/manual/zh/function.error-log.php) などの関数を利用して個別に実装することができます。

## 例

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$logFile = '/tmp/workerman.log';

$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start";
};
// workerを実行する
Worker::runAll();
```
