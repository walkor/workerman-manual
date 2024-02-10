# Yapıcı Fonksiyon __construct

## Açıklama:
```php
Worker::__construct([string $dinle , array $context])
```

Worker konteyner örneğini başlatır ve belirli bir işlevi tamamlamak için konteynerin bazı özelliklerini ve geri çağrı arabirimlerini ayarlayabilir.


## Parametreler
#### **``` $dinle ```** (isteğe bağlı, belirtilmezse herhangi bir bağlantı noktası dinlenmediği anlamına gelir.)



Eğer ```$dinle``` parametresi belirtilmişse, soket dinlemesi yapılır.

$listen formatı <protokol>://<dinleme adresi>

**<protokol> aşağıdaki formatlardan biri olabilir:**


tcp: Örneğin ```tcp://0.0.0.0:8686```

udp: Örneğin ```udp://0.0.0.0:8686```

unix: Örneğin ```unix:///tmp/my_file``` ```(Workerman>=3.2.7 gereklidir)```

http: Örneğin ```http://0.0.0.0:80```

websocket: Örneğin ```websocket://0.0.0.0:8686```

text: Örneğin ```text://0.0.0.0:8686``` ```(text, Workerman tarafından yerleşik bir metin protokolüdür, telnet ile uyumludur, ayrıntılar için Ek Text protokolüne bakınız)```


Ve diğer özel protokoller, bu kılavuzun özel iletişim protokolü bölümüne bakınız.


**<dinleme adresi> aşağıdaki formatlardan biri olabilir:**

Eğer unix soket ise, adres yerel disk yoludur.

Unix soketi değilse, adres formatı <host IP>:<port>

<host IP> ```0.0.0.0``` yerel tüm ağ kartlarını içeren, dahili IP'leri, harici IP'leri ve yerel döngüyü içerir.

<host IP>```127.0.0.1``` yerel döngüyü dinleyip sadece yerel makineden erişilebilir, dış dünyadan erişilemez.

<host IP> ```192.168.xx.xx``` gibi dahili IP adresleri, yalnızca dahili IP'leri dinler, harici kullanıcılar erişemez.

Belirtilen <host IP>, mevcut bir host IP'si değilse dinleyemez ve ```Cannot assign requested address``` hatası verir.

**Not:** <port> 65535'ten büyük olamaz. <port> 1024'ten küçükse soketi dinleyebilmek için kök yetki gereklidir. Dinlenen port mevcut olmayan bir makineye ait olmalıdır, aksi takdirde dinlenemez ve ```Address already in use``` hatası verir.



#### **``` $context ```**


Bir dizi. Soketin bağlam seçeneklerini iletmek için kullanılır, [Socket Context Options](https://php.net/manual/tr/context.socket.php) bakınız.


## Örnekler

Worker HTTP isteklerini işlemek üzere HTTP konteyneri olarak dinler
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send("merhaba");
};

// Worker'ı çalıştır
Worker::runAll();
```


Worker websocket isteklerini işlemek üzere websocket konteyneri olarak dinler
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("merhaba");
};

// Worker'ı çalıştır
Worker::runAll();
```


Worker TCP isteklerini işlemek üzere TCP konteyneri olarak dinler
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("merhaba");
};

// Worker'ı çalıştır
Worker::runAll();
```


Worker UDP isteklerini işlemek üzere UDP konteyneri olarak dinler
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("merhaba");
};

// Worker'ı çalıştır
Worker::runAll();
```


Worker unix domain soketini dinler ```(Workerman sürümü >=3.2.7 gerektirir)```
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('unix:///tmp/my.sock');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("merhaba");
};

// Worker'ı çalıştır
Worker::runAll();
```


Herhangi bir dinleme yapmayan Worker konteyneri, bazı zamanlanmış görevleri işlemek için kullanılır
```php
use \Workerman\Worker;
use \Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function($task)
{
    // Her 2.5 saniyede bir çalıştır
    $time_interval = 2.5;
    Timer::add($time_interval, function()
    {
        echo "görev çalıştır\n";
    });
};

// Worker'ı çalıştır
Worker::runAll();
```


**Worker özel bir protokolü dinler**

Son dizin yapısı
```
├── Protocols              // Oluşturulmak istenen Protocols klasörü
│   └── MyTextProtocol.php // Oluşturulmak istenen özel protokol dosyası
├── test.php  // Oluşturulmak istenen test betiği
└── Workerman // Workerman kaynak kod dizini, buradaki kodlar değişmiyor
```

1. Protocols dizini oluşturun ve bir protokol dosyası oluşturun
Protocols/MyTextProtocol.php (yukarıdaki dizin yapısına bakınız)

```php
// Kullanıcı özel protokol ad alanı Protocols olarak ayarlanmalıdır
namespace Protocols;
// Basit metin protokolü, protokol formatı metin + satır başıdır
class MyTextProtocol
{
    // Paketleme işlevi, mevcut paketin uzunluğunu döndürür
    public static function input($recv_buffer)
    {
        // Satır sonu bulunur
        $pos = strpos($recv_buffer, "\n");
        // Satır sonu bulunamazsa, tam bir paket değil, 0 döndürerek veri bekler
        if($pos === false)
        {
            return 0;
        }
        // Satır sonu bulunursa, mevcut paketin uzunluğunu, satır sonu dahil döndürür
        return $pos+1;
    }

    // Tam paket alındıktan sonra otomatik olarak MyTextProtocol::decode('alınan_data') çalışır
    // Sonuç $data yoluyla onMessage geri çağrısına aktarılır
    public static function decode($recv_buffer)
    {
        return trim($recv_buffer);
    }

    // Sunucu tarafından gönderilmeden önce data encode edilir, burada satır sonu eklenir
    public static function encode($data)
    {
        return $data."\n";
    }
}
```
2. MyTextProtocol protokolünü kullanarak istekleri dinleme

Yukarıdaki son dizin yapısına bakarak test.php dosyasını oluşturun

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// #### MyTextProtocol worker ####
$text_worker = new Worker("MyTextProtocol://0.0.0.0:5678");

/*
 * Tam veri alındıktan sonra (satır sonuyla biten) otomatik MyTextProtocol:: decode('alınan veri') çalışır
 * Sonuç $data yoluyla onMessage'e iletilir
 */
$text_worker->onMessage =  function(TcpConnection $connection, $data)
{
    var_dump($data);
    /*
     * Client'a veri gönderilirken, otomatik olarak MyTextProtocol:: encode('hello world') kodlanır
     * ve sonra istemciye gönderilir
     */
    $connection->send("hello world");
};

// Tüm çalışanları çalıştır
Worker::runAll();
```


3. Test

Terminali açın, test.php dosyasının bulunduğu dizine gidin ve ```php test.php start``` komutunu çalıştırın
```php test.php start
Workerman[test.php] DEBUG modunda başlatıldı
----------------------- WORKERMAN -----------------------------
Workerman sürümü: 3.2.7          PHP sürümü:5.4.37
------------------------ WORKERS --------------------------------
kullanıcı          işçi         dinleme                         süreçler durum
root          yok          myTextProtocol://0.0.0.0:5678   1         [OK]
----------------------------------------------------------------
Çıkmak için Ctrl-C'ye basın. Başlatma başarılı.
```

Terminali açın ve telnet kullanarak test edin (linux sistemlerinin telnet kullanması önerilir)

Varsayılan olarak yerel IP için test edildiği düşünülürse, terminalde telnet 127.0.0.1 5678 komutunu çalıştırın
ve ardından hi yazarak enter tuşuna basın
```php
telnet 127.0.0.1 5678
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
hi
hello world
```
