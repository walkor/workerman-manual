```php
int \Workerman\Timer::sleep(float $delay)
```
PHPの組み込みsleep()関数に似ていますが、違いはTimer::sleep()がブロックされないことです（現在のプロセスをブロックしません）

> **注意**
> この機能にはworkerman>=5.0が必要です
> この機能にはcomposer require revolt/event-loop ^1.0.0をインストールする必要があります。またはSwoole/Swowをイベント駆動として使用することもできます。

### パラメータ
``` delay ```

1回の実行までの時間（秒単位）、小数もサポートされており、0.001秒（ミリ秒レベル）まで精確です。

### 戻り値
戻り値はありません。

### 例

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    // 1.5秒遅延してデータを送信
    Timer::sleep(1.5);
    // データを送信
    $connection->send('hello workerman');
};

Worker::runAll();
```
