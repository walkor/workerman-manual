16進数データの送受信方法

**16進数データの受信**

データを受信した後、```bin2hex($data)```関数を使用してデータを16進数に変換することができます。

**16進数データの送信**

データを送信する前に、```hex2bin($data)```を使用して16進数データをバイナリに変換して送信します。

**例：**

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:8080');
$worker->onMessage = function(TcpConnection $connection, $data){
    // 16進数データを取得する
    $hex_data = bin2hex($data);
    // クライアントに16進数データを送信する
    $connection->send(hex2bin($hex_data));
};
Worker::runAll();
```
