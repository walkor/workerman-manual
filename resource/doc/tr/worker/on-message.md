# onMessage
## Açıklama:
```php
geri çağırma İşçi::$onMessage
```
Müşteri sunucu üzerinden veri gönderdiğinde (Workerman veri aldığında) tetiklenen geri çağırma işlevi

## Geri çağırma işlevinin parametreleri

``` $connection ```

Bağlantı nesnesi, yani [TcpConnection örneği](../tcp-connection.md), müşteri bağlantısını işlemek için [veri gönderme](../tcp-connection/send.md), [bağlantıyı kapatma](../tcp-connection/close.md) gibi işlevler için kullanılır.

``` $data ```

Müşteri bağlantısından gelen veri, eğer İşçi bir protokol belirtmişse, $data ilgili protokolün decode (çözme) edilmiş verisidir. Veri türü ve protokol `decode()` uygulamasıyla ilgilidir, `websocket` `text` `frame` için string, HTTP protokolü için [`Workerman\Protocols\Http\Request`](../http/request.md) nesnesidir.

## Örnek

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($data);
    $connection->send('alım başarılı');
};
// İşçiyi çalıştır
Worker::runAll();
```

Not: Geri çağırma olarak anonim işlev kullanmanın yanı sıra, diğer geri çağırma yazımı için [buraya bakabilirsiniz](../faq/callback_methods.md).
