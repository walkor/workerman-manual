# 解除購読
**```（Workermanバージョン>=3.3.0が必要）```**

```php
void \Channel\Client::unsubscribe(string $event_name)
```

特定のイベントの購読を解除します。このイベントが発生した場合、登録された```on($event_name, $callback)```のコールバック```$callback```はもはやトリガーされません。

### パラメータ
``` $event_name ```

イベント名

### 戻り値
void

### 例
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->onWorkerStart = function()
{
    Channel\Client::connect('127.0.0.1', 2206);
    $event_name = 'user_login';
    Channel\Client::on($event_name, function($event_data){
        var_dump($event_data);
    });
    Channel\Client::unsubscribe($event_name);
};

Worker::runAll();
```
