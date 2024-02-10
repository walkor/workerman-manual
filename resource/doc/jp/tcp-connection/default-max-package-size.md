# maxPackageSize

## 説明:
```php
static int Connection::$defaultMaxPackageSize
```

この属性はグローバルな静的プロパティであり、各接続が受信できる最大パケットサイズを設定するために使用されます。デフォルトでは10MBです。

送信されるデータパケットが解析された場合(プロトコルクラスのinputメソッドの戻り値)、そのパケットの長さが```Connection::$defaultMaxPackageSize```よりも大きい場合、不正なデータと見なされ、接続が切断されます。

## 例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 各接続が受信するデータパケットの最大サイズを1024000バイトに設定
TcpConnection::$defaultMaxPackageSize = 1024000;

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello');
};
// ワーカーを実行
Worker::runAll();
```
