# onConnect
## Açıklama:
```php
geri çağırma Worker::$onConnect
```

İstemci, Workerman ile bağlantı kurduğunda (TCP üçlü el sıkışma tamamlandığında) tetiklenen geri çağırma fonksiyonu. Her bağlantı yalnızca bir kez ```onConnect``` geri çağırmasını tetikler.

Not: onConnect etkinliği yalnızca istemcinin Workerman ile TCP üçlü el sıkışmayı tamamlamasını temsil eder, bu sırada istemciden herhangi bir veri gelmemiş olabilir. Bu nedenle, onConnect etkinliği sırasında istemcinin kim olduğunu doğrulamak imkansızdır. Karşı tarafın kim olduğunu bilmek istiyorsanız, istemcinin kimliğini doğrulamak için, örneğin bir token veya kullanıcı adı şifre gibi kimlik doğrulama verilerini göndermesi gerekir, bu bilgiler [onMessage geri çağırmasında](on-message.md) doğrulama yapılabilir.

UDP bağlantısı kullanıldığında, onConnect geri çağrması tetiklenmeyecek ve onClose geri çağrması da tetiklenmeyecektir.

## Geri çağırma fonksiyonunun parametreleri

 ``` $connection ```

Bağlantı nesnesi, yani [TcpConnection örneği](../tcp-connection.md), istemci bağlantısını işlemek için [veri gönderme](../tcp-connection/send.md), [bağlantıyı kapatma](../tcp-connection/close.md) gibi işlemler için kullanılır.


## Örnek

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "ip adresinden yeni bağlantı " . $connection->getRemoteIp() . "\n";
};
// Worker'ı çalıştır
Worker::runAll();
```

Not: Geri çağırma olarak anonim fonksiyon kullanmanın yanı sıra, [buraya bakarak](../faq/callback_methods.md) diğer geri çağırma yazma yöntemlerini de kullanabilirsiniz.
