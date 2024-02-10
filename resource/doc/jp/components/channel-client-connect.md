# 接続
**```（要求Workermanバージョン>=3.3.0）```**
```php
void \Channel\Client::connect([string $listen_ip = '127.0.0.1', int $listen_port = 2206])
```
Channel/Serverに接続します。

### パラメーター
 ``` listen_ip ```

Channel/Serverが監視するIPアドレス。デフォルトは```127.0.0.1```です。

 ``` listen_port ```

Channel/Serverが監視するポート。デフォルトは2206です。


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
};

Worker::runAll();
```
