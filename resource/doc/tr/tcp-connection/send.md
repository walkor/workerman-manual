# send
## Açıklama:
```php
mixed Connection::send(mixed $data [,$raw = false])
```

İstemciye veri gönderir.

## Parametreler

``` $data ```

Gönderilmek istenen veri. Eğer Worker sınıfını başlatırken bir protokol belirtildiyse, otomatik olarak protokolün encode yöntemini çağırarak protokol paketleme işlemini tamamlar ve istemciye gönderir.

``` $raw ```

Ham veri gönderilsin mi, yani protokolün encode yöntemini çağırmayacak mı, varsayılan olarak false'tur, yani protokolün encode yöntemini otomatik olarak çağırır.

## Dönüş Değeri

true: Veriler bu bağlantının işletim sistemi seviyesindeki soket gönderme önbelleğine başarıyla yazıldı.

null: Veriler bu bağlantının uygulama seviyesindeki gönderme önbelleğine yazıldı ve işletim sistemi seviyesindeki soket gönderme önbelleğine yazılmasını bekliyor.

false: Gönderme başarısız olduğunu gösterir, başarısızlık nedeni istemci bağlantısının kapanmış olması veya bu bağlantının uygulama seviyesindeki gönderme önbelleğinin dolmuş olması olabilir.

## Notlar
send ```true``` döndüğünde, sadece verilerin bu bağlantının işletim sistemi seviyesindeki soket gönderme önbelleğine başarıyla yazıldığını ifade eder; verilerin karşı soketin alım önbelleğine başarıyla gönderildiği anlamına gelmez ve daha da önemlisi karşı uygulama programının yerel soket alım önbelleğinden veri okuduğu anlamına gelmez. **Ancak, eğer send false dönmezse ve ağ bağlantısı kesilmezse ve istemci normal şekilde veri alıyorsa, verinin genellikle karşı tarafa %100 gönderildiğini söyleyebiliriz.**

Soket gönderme önbelleğindeki verilerin işletim sistemi tarafından karşı tarafa asenkron olarak gönderildiğini ve işletim sisteminin uygulama seviyesine herhangi bir onay mekanizması sağlamadığını unutmayın, bu yüzden **uygulama seviyesinde** verilerin ne zaman gönderilmeye başladığını **uygulama seviyesinde** ne zaman gönderildiğini bilemez. **Uygulama seviyesi** verilerin ne zaman gönderildiğini veya başarılı bir şekilde gönderilip gönderilmediğini bilmez. Bu nedenle workerman, doğrudan mesaj onaylama arabirimi sağlayamaz.

Eğer işletme her mesajın istemciye ulaşmasını garanti altına almak istiyorsa, bir onay mekanizması eklemelidir. Onay mekanizması işletmeye ve gereksinimlere göre değişebilir, aynı işlev için birkaç farklı yöntem olabilir.

Örneğin, sohbet sistemi için şu tür bir onay mekanizması kullanılabilir. Her mesaj veritabanına kaydedilir, her mesajda bir okundu mu alanı bulunur. İstemci her bir mesaj aldığında sunucuya bir onay paketi gönderir, sunucu ilgili mesajı okundu olarak işaretler. İstemci sunucuya bağlandığında (genellikle kullanıcı giriş yaptığında veya yeniden bağlanma durumunda), okunmamış mesaj olup olmadığını sorgular, eğer varsa sunucu istemciye gönderir, ve aynı şekilde istemci veri aldığında sunucuya okundu olarak işaretler. Bu şekilde her mesajın karşı tarafa ulaşmasını sağlanabilir. Elbette, geliştirici kendi onay mantığını da kullanabilir.

## Örnek

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Otomatik olarak \Workerman\Protocols\Websocket::encode özelliğini kullanarak websocket protokol verisine paketleyen bir işlem yapılır ve gönderilir.
    $connection->send("hello\n");
};
// Worker çalıştırılır.
Worker::runAll();
```
