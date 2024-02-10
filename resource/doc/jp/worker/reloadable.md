# reloadable
## 説明:
```php
bool Worker::$reloadable
```

`php start.php reload`を実行すると、すべての子プロセスにreloadシグナル(SIGUSR1)が送信されます。

子プロセスはreloadシグナルを受信すると自動的に終了し、その後親プロセスが新しいプロセスを自動的に起動します。通常、これはビジネスコードを更新するために使用されます。

プロセスの$reloadableがfalseの場合、reloadシグナルを受信しても [onWorkerReload](on-worker-reload.md) がトリガされるだけで、現在のプロセスは再起動されません。

例えば、Gateway/Workerモデルでは、gatewayプロセスがクライアント接続を維持し、workerプロセスがリクエストを処理する役割を担っています。
gatewayプロセスのreloadable属性をfalseに設定すると、クライアント接続を切断することなくビジネスコードを更新できます。

## 例

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// このインスタンスがreloadシグナルを受信した後に再起動するかどうかを設定する
$worker->reloadable = false;
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};
// workerを実行する
Worker::runAll();
```
