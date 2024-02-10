# 16'luk veri nasıl gönderilir ve alınır

**16'luk veri almak**

Veri alındıktan sonra, `bin2hex($data)` fonksiyonu kullanılarak veri 16'luk sisteme dönüştürülebilir.

**16'luk veri göndermek**

Veri göndermeden önce, `hex2bin($data)` fonksiyonu kullanılarak 16'luk veri ikili sisteme dönüştürülür ve gönderilir.

**Örnek:**

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:8080');
$worker->onMessage = function(TcpConnection $connection, $data){
    // 16'luk veriyi al
    $hex_data = bin2hex($data);
    // İstemciye 16'luk veri gönder
    $connection->send(hex2bin($hex_data));
};
Worker::runAll();
```
