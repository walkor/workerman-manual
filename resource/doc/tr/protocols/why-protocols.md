# İletişim Protokolünün Rolü
TCP'nin akış temelli olması nedeniyle istemci tarafından gönderilen istek verileri, su gibi akarak sunucuya girer. Sunucu gelen verilerin tamamının geldiğini kontrol etmelidir, çünkü belki sadece bir isteğin bir kısmı sunucuya ulaşmış olabilir veya hatta birden fazla istek bir arada sunucuya ulaşabilir. İsteklerin tamamının ulaşıp ulaşmadığını veya bir arada gelen isteklerin nasıl ayrılacağını belirlemek için bir iletişim protokolü belirlenmesi gerekmektedir.

## WorkerMan'de Neden Bir Protokol Belirlemek Gereklidir?
Geleneksel PHP geliştirme genellikle Web'e dayalıdır ve genellikle HTTP protokolü üzerinden gerçekleşir. HTTP protokolünün ayrıştırılma işlemleri genellikle Web Sunucusu tarafından tek başına gerçekleştirildiği için geliştiriciler genellikle protokol ile ilgilenmezler. Ancak HTTP dışında bir protokol üzerinden geliştirme yapılması gerektiğinde, geliştiricilerin protokol ile ilgilenmesi gerekmektedir.

## WorkerMan Tarafından Desteklenen Protokoller
WorkerMan şu anda HTTP, websocket, text protokolünü (ek bilgilere bakınız), frame protokolünü (ek bilgilere bakınız), ws protokolünü (ek bilgilere bakınız) desteklemektedir. Bu protokoller üzerinden iletişim kurmak gerektiğinde doğrudan kullanılabilir. Kullanımı şu şekildedir: Worker başlatılırken protokol belirtilmelidir, örneğin

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// websocket://0.0.0.0:2345 2345 portunda websocket protokolü ile dinleme yapılacağını belirtir
$websocket_worker = new Worker('websocket://0.0.0.0:2345');

// text protokolü
$text_worker = new Worker('text://0.0.0.0:2346');

// frame protokolü
$frame_worker = new Worker('frame://0.0.0.0:2347');

// TCP Worker, doğrudan soket ile iletişim sağlar, herhangi bir uygulama katmanı protokolü kullanmaz
$tcp_worker = new Worker('tcp://0.0.0.0:2348');

// UDP Worker, herhangi bir uygulama katmanı protokolü kullanmaz
$udp_worker = new Worker('udp://0.0.0.0:2349');

// Unix domain Worker, herhangi bir uygulama katmanı protokolü kullanmaz
$unix_worker = new Worker('unix:///tmp/wm.sock');

```

## Özel İletişim Protokolü Kullanımı
WorkerMan tarafından sağlanan iletişim protokolleri geliştirme ihtiyaçlarını karşılayamadığında, geliştiriciler kendi özel iletişim protokollerini özelleştirebilir. Özelleştirme yöntemi bir sonraki bölümde açıklanmaktadır.

**Not:**

Workerman, bir text protokolü de dahil olmak üzere dahili olarak birçok protokol sunar; bu protokol metin ve satırbaşı karakterlerden oluşur. text protokolü geliştirme ve hata ayıklama işlemleri oldukça basittir, çoğu özel protokol senaryosu için uygun olup, telnet hata ayıklamayı destekler. Geliştiriciler, kendi uygulama protokollerini geliştirmek istiyorlarsa, ayrıca geliştirmeye gerek kalmadan doğrudan text protokolünü kullanabilirler.

text protokolü için açıklama "Ekler Metin Protokolü Bölümü"ne bakınız.
