# onClose
## Açıklama:
```php
geriÇağırma Worker::$onClose
```

Client bağlantısı Workerman'dan koparsa tetiklenen geri çağırma fonksiyonu. Bağlantının nasıl koptuğuna bakılmaksızın, koptuğunda ```onClose``` tetiklenecektir. Her bağlantı sadece bir kez ```onClose``` tetikler.

Not: Karşı taraftaki bağlantı, ağ kesilmesi veya güç kesilmesi gibi aşırı durumlarda koparsa, Workerman'a zamanında tcp fin paketi gönderilemediği için Workerman bağlantının koptuğunu bilemeyecek ve zamanında ```onClose``` tetikleyemeyecektir. Bu durumu uygulama katmanındaki kalp atışıyla çözmek gerekmektedir. Workerman'da bağlantı kalp atışının nasıl yapıldığına [buradan](../faq/heartbeat.md) bakabilirsiniz. GatewayWorker çerçevesi kullanılıyorsa, doğrudan GatewayWorker çerçevesinin kalp atış mekanizmasını kullanabilirsiniz, [buradan](https://doc2.workerman.net/heartbeat.html) bakınız.

UDP bağlantısı bağlantısız olduğu için UDP kullanırken ```onConnect``` geri çağırma tetiklenmez ve aynı şekilde ```onClose``` geri çağırma da tetiklenmez.

## Geri çağırma fonksiyonunun parametreleri

 ``` $connection ```

Bağlantı nesnesi, yani [TcpConnection örneği](../tcp-connection.md), müşteri bağlantısını işlemek için [veri göndermek](../tcp-connection/send.md), [bağlantıyı kapatmak](../tcp-connection/close.md) gibi işlemler yapmak için kullanılır.

## Örnek

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onClose = function(TcpConnection $connection)
{
    echo "bağlantı kapatıldı\n";
};
// Worker'ı çalıştır
Worker::runAll();
```

Not: Geri çağırma olarak anonim işlevi kullanmanın yanı sıra, diğer geri çağırma kodlama yöntemleri için [buradan](../faq/callback_methods.md) bakılabilir.
