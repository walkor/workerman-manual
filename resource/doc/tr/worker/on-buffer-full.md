# onBufferFull
## Açıklama:
```php
callback Worker::$onBufferFull
```

Her bağlantıda ayrı bir uygulama katmanı gönderi tamponu bulunmaktadır. Eğer istemci tarafının alım hızı, sunucu tarafının gönderme hızından daha düşükse, veriler uygulama katmanı tamponunda bekletilir ve tampon dolarsa onBufferFull geri çağrısı tetiklenir.

Tampon boyutu [TcpConnection::$maxSendBufferSize](../tcp-connection/max-send-buffer-size.md)'e bağlı olarak değişir, varsayılan değeri 1MB'dir ve mevcut bağlantı için tampon boyutunu dinamik olarak ayarlayabilirsiniz, örneğin:
```php
// Mevcut bağlantı için gönderme tampon boyutunu belirle, birim byte
$connection->maxSendBufferSize = 102400;
```
Ayrıca, [TcpConnection::$defaultMaxSendBufferSize](../tcp-connection/default-max-send-buffer-size.md) kullanarak tüm bağlantılar için varsayılan tampon boyutunu ayarlayabilirsiniz, örneğin:
```php
use Workerman\Connection\TcpConnection;
// Tüm bağlantılar için varsayılan uygulama katmanı gönderi tampon boyutunu belirle, birim byte
TcpConnection::$defaultMaxSendBufferSize = 2*1024*1024;
```

Bu geri çağrının, bağlantıya Connection::send çağrısı yaptıktan hemen sonra tetiklenebilir. Örneğin, büyük veri gönderimi veya ardışık olarak hızlı bir şekilde veri gönderimi durumlarında, ağ veya diğer nedenlerden dolayı veriler karşı bağlantının gönderme tamponunda birikir ve TcpConnection::$maxSendBufferSize sınırını aştığında tetiklenir.

onBufferFull etkinliği meydana geldiğinde, geliştiricinin genellikle, karşı tarafa veri göndermeyi durdurmak gibi önlemler alması gerekebilir ve gönderme tamponundaki verilerin gönderilmesini beklemesi (onBufferDrain etkinliği) gerekebilir.

Connection::send(`$A`) çağrısı yapıldığında onBufferFull etkinliği tetiklendiğinde, o anda gönderilen veri `$A` ne kadar büyük olursa olsun, hatta TcpConnection::$maxSendBufferSize'dan daha büyük olsa bile, gönderilecek veri hala gönderme tamponuna yerleştirilecektir. Yani gönderme tamponuna gerçekten yerleştirilen veri, TcpConnection::$maxSendBufferSize'dan çok daha büyük olabilir. Gönderme tamponu zaten TcpConnection::$maxSendBufferSize'dan büyük olduğunda Connection::send(`$B`) çağrısı yapıldığında, bu $B verisi gönderme tamponuna yerleştirilmeyecek, bunun yerine atılacak ve onError geri çağrısını tetikleyecektir.

Özetle, gönderme tamponu henüz dolmadıysa, hatta bir byte boş alan kalsa bile, Connection::send(`$A`) çağrısı kesinlikle `$A`yı gönderme tamponuna yerleştirecektir. Gönderme tamponuna yerleştikten sonra, gönderme tamponu boyutu TcpConnection::$maxSendBufferSize sınırını aşıyorsa, onBufferFull geri çağrısı tetiklenecektir.

## Geri çağrı fonksiyonun parametreleri

 ``` $connection ```

Bağlantı nesnesi, yani [TcpConnection örneği](../tcp-connection.md), istemci bağlantısını yönetmek için [veri gönderme](../tcp-connection/send.md), [bağlantıyı kapatma](../tcp-connection/close.md) gibi işlemler için kullanılır.


## Örnek

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onBufferFull = function(TcpConnection $connection)
{
    echo "Tampon dolu ve tekrar gönderme yapılmaz\n";
};
// Worker çalıştırma
Worker::runAll();
```

Not: Geri çağrı olarak anonim fonksiyon kullanmaktan başka, diğer geri çağrı yazma yöntemleri için [buraya bakabilirsiniz](../faq/callback_methods.md).

## Ayrıca bakınız
onBufferDrain - Bağlantının uygulama katmanı gönderi tamponundaki tüm veriler gönderildiğinde tetiklenir
