# Channelコンポーネントサーバー

**``` (Workermanのバージョン>=3.3.0が必要です) ```**

# __construct
```php
void \Channel\Server::__construct([string $listen_ip = '0.0.0.0', int $listen_port = 2206])
```

\Channel\Serverサーバーをインスタンス化します。

### パラメーター
 ``` listen_ip ```

リッスンするローカルIPアドレス、デフォルトでは```0.0.0.0```です。

 ``` listen_port ```

リッスンするポート番号、デフォルトでは2206です。

## 例

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// パラメーターを指定しないと、デフォルトで0.0.0.0:2206をリッスンします
$channel_server = new Channel\Server();

if(!defined('GLOBAL_START'))
{
    Worker::runAll();
}
```
