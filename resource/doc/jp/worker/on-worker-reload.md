# onWorkerReload
要求 ```（workerman >= 3.2.5）```
## 説明：
```php
callback Worker::$onWorkerReload
```
この機能はあまり使われません。

Workerがreloadシグナルを受信した後に実行されるコールバックを設定します。

onWorkerReloadコールバックを使用して、プロセスを再起動することなく、ビジネス設定ファイルを再読み込みするなどのことができます。

**注意**：

子プロセスがreloadシグナルを受信した後、デフォルトの動作は終了して再起動することです。これにより新しいプロセスがビジネスコードを再読み込んで更新を完了します。したがって、reload後、子プロセスがonWorkerReloadコールバックを実行した後にすぐに終了するのは正常な動作です。

reloadシグナルを受信した後、子プロセスがonWorkerReloadのみを実行し、終了しないようにしたい場合は、対応するWorkerインスタンスのreloadableプロパティをfalseに設定して初期化することができます。


## コールバック関数のパラメータ

 ``` $worker ```

Workerオブジェクト



## 例


```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// reloadableをfalseに設定し、つまり子プロセスがreloadシグナルを受信した場合に再起動を実行しない
$worker->reloadable = false;
// reload後にすべてのクライアントにサーバーがreloadを実行したことを通知する
$worker->onWorkerReload = function(Worker $worker)
{
    foreach($worker->connections as $connection)
    {
        $connection->send('worker reloading');
    }
};
// ワーカーを実行
Worker::runAll();
```

ヒント：コールバックとして匿名関数を使用する代わりに、[こちら](../faq/callback_methods.md)でその他のコールバックの書き方を参照することもできます。
