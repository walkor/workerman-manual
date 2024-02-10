# ユーザー

## 説明:
```php
string Worker::$user
```

現在のWorkerインスタンスをどのユーザーで実行するかを設定します。このプロパティは、現在のユーザーがrootである場合にのみ有効です。設定しない場合、デフォルトで現在のユーザーで実行されます。

```$user``` には、www-data、apache、nobodyなどの権限の低いユーザーを設定することをお勧めします。

注意：このプロパティは、```Worker::runAll();``` を実行する前に設定する必要があります。Windowsシステムではこの機能はサポートされていません。

## 例

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// インスタンスの実行ユーザーを設定
$worker->user = 'www-data';
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};
// ワーカーを実行
Worker::runAll();
```
