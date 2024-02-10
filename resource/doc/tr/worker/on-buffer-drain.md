# onBufferDrain
## Açıklama:
```php
geriçağırma işlevi Worker::$onBufferDrain
```

Her bağlantının ayrı bir uygulama katmanı gönderi tamponu bulunmaktadır ve tampon boyutu ```TcpConnection::$maxSendBufferSize``` tarafından belirlenir, varsayılan değer 1MB'dir ve manuel olarak değiştirilerek değişikliğin tüm bağlantılara etki etmesi sağlanabilir.

Bu geriçağırma, uygulama katmanı gönderi tamponu verilerinin tamamının gönderildikten sonra tetiklenir. Genellikle onBufferFull ile birlikte kullanılır, örneğin onBufferFull'de karşı tarafa veri göndermeyi durdurmak, onBufferDrain'de veri yazmayı kaldırma işlemi gibi.

## Geriçağırma işlevi parametreleri

 ``` $connection ```

Bağlantı nesnesi, yani [TcpConnection örneği](../tcp-connection.md), istemci bağlantısını işlemek için kullanılır, örneğin [veri gönderme](../tcp-connection/send.md), [bağlantıyı kapatma](../tcp-connection/close.md) vb.

## Örnek

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onBufferFull = function(TcpConnection $connection)
{
    echo "tamponDolu ve tekrar gönderme\n";
};
$worker->onBufferDrain = function(TcpConnection $connection)
{
    echo "tampon boşaltma ve göndermeye devam etme\n";
};
// Worker çalıştır
Worker::runAll();
```

Not: Geriçağırma olarak anonim işlevler kullanmanın dışında, başka geriçağırma yazımı için  [buraya bakabilirsiniz](../faq/callback_methods.md).

## Ayrıca bakınız
onBufferFull: Bağlantının uygulama katmanı gönderi tamponu dolu olduğunda tetiklenir
