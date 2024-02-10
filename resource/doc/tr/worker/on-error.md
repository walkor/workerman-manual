# onError
## Açıklama:
```php
geriçağırım Worker::$onError
```
Müşteri bağlantısında bir hata oluştuğunda tetiklenir.

Şu anda hata türleri şunlardır

1. Connection::send'i aramak başarısız olduğunda (müşteri bağlantısı kesildiği için) (hemen onClose geriçağırımını tetikler) ```(kod:WORKERMAN_SEND_FAIL msg:müşteri kapandı)```

2. onBufferFull'ü tetikledikten sonra (gönderme tamponu dolu) ve hala Connection::send'i çağırırsak ve gönderme tamponu hala doluysa gönderme başarısız olur (onClose geri çağrımını tetiklemez)```(kod:WORKERMAN_SEND_FAIL msg:gönderme tamponu dolu ve paket düşürüldü)```

3. AsyncTcpConnection asenkron bağlantı hatası durumunda (hemen onClose geri çağırımını tetikler) ```(kod:WORKERMAN_CONNECT_FAIL msg:stream_socket_client tarafından döndürülen hata mesajı)```


## Geriçağırım işlevi parametreleri

 ``` $connection ```

Bağlantı nesnesi, yani [TcpConnection örneği](../tcp-connection.md), müşteri bağlantısını yönetmek için kullanılır, örneğin [veri gönderimi](../tcp-connection/send.md), [bağlantıyı kapatma](../tcp-connection/close.md) vb.

 ``` $code ```

Hata kodu

 ``` $msg ```

Hata iletileri


## Örnek

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onError = function(TcpConnection $connection, $code, $msg)
{
    echo "hata $code $msg\n";
};
// worker'ı çalıştır
Worker::runAll();
```

Not: Geriçağırım olarak anonim fonksiyonun yanı sıra [buradan referans alarak](../faq/callback_methods.md) diğer geri çağırım yazma yöntemlerini de kullanabilirsiniz.
