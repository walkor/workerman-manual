# GlobalData コンポーネントのサーバー
**``` (Workermanのバージョン>=3.3.0が必要です) ```**

# __construct
```php
void \GlobalData\Server::__construct([string $listen_ip = '0.0.0.0', int $listen_port = 2207])
```

\GlobalData\Serverサービスをインスタンス化します

### パラメーター
 ``` listen_ip ```

監視するローカルIPアドレス、デフォルトは```0.0.0.0```です

 ``` listen_port ```

監視するポート番号、デフォルトは2207です

## 例
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// ポートを監視
$worker = new GlobalData\Server('127.0.0.1', 2207);

Worker::runAll();
```
