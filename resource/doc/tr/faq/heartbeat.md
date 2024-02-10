# Kalp Atışı

Not: Uzun süreli bağlantı uygulamalarında mutlaka bir kalp atışı eklenmelidir, aksi halde iletişim eksikliği nedeniyle bağlantı kesilebilir.

Kalp atışının temel amacı şunlardır:

1. İstemci düzenli aralıklarla sunucuya veri göndererek iletişim eksikliğinden dolayı bazı ağ noktalarının güvenlik duvarları tarafından kapatılmasını engeller ve bağlantının kesilmesine neden olur.

2. Sunucu, kalp atışı sayesinde istemcinin çevrimiçi olup olmadığını belirleyebilir. Belirli bir süre içinde istemciden herhangi bir veri gelmezse istemciyi çevrimdışı kabul eder. Bu şekilde, istemcinin aşırı durumlarda (elektrik kesintisi, ağ kesintisi vb.) çevrimdışı olmasını tespit edebilir.

Kalp atışı aralığı için önerilen değerler:

İstemcinin kalp atışını 60 saniyeden az bir sürede göndermesi önerilir, örneğin 55 saniye.

> Kalp atışının veri formatı belirli bir formatta olmak zorunda değildir, sunucu tarafından tanınabilmesi yeterlidir.

## Kalp Atışı Örneği
```php
<?php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Kalp atışı 55 saniyede bir gönderilir
define('HEARTBEAT_TIME', 55);

$worker = new Worker('text://0.0.0.0:1234');

$worker->onMessage = function(TcpConnection $connection, $msg) {
    // Bağlantıya son mesajın alındığı zamanı kaydetmek için geçici olarak bir lastMessageTime özelliği ayarla
    $connection->lastMessageTime = time();
    // Diğer iş mantığı...
};

// İşçi başlatıldıktan sonra her 10 saniyede bir çalışacak bir zamanlayıcı ayarla
$worker->onWorkerStart = function($worker) {
    Timer::add(10, function() use ($worker){
        $time_now = time();
        foreach($worker->connections as $connection) {
            // Bu bağlantının henüz bir mesaj almadığı olasılığı vardır, bu durumda lastMessageTime şu anki zaman olarak ayarlanır
            if (empty($connection->lastMessageTime)) {
                $connection->lastMessageTime = $time_now;
                continue;
            }
            // Son iletişim zamanı aralığı kalp atışı aralığından büyükse, istemcinin çevrimdışı olduğunu kabul ederek bağlantıyı kapat
            if ($time_now - $connection->lastMessageTime > HEARTBEAT_TIME) {
                $connection->close();
            }
        }
    });
};

Worker::runAll();
```

Yukarıdaki yapılandırmaya göre, istemci 55 saniye boyunca sunucuya herhangi bir veri göndermezse, sunucu istemciyi çevrimdışı kabul eder, bağlantıyı kapatır ve onClose olayını tetikler.

## Bağlantıya Yeniden Bağlanma (Önemli)

İstemci kalp atışı gönderse veya sunucu kalp atışı gönderse, bağlantının kopma olasılığı her zaman vardır. Örneğin tarayıcı penceresi küçültüldüğünde JavaScript durdurulur, tarayıcı başka bir sekmele geçtiğinde JavaScript durdurulur, bilgisayar uykuya geçtiğinde, mobil cihazdaki ağ değiştiğinde, sinyal zayıfladığında, telefon ekranı kapandığında, telefon uygulaması arka plana geçtiğinde, yönlendirici arızalandığında, işletme tarafından bilinçli olarak kapatma yapıldığında gibi durumlarda bağlantı kopabilir. Özellikle dış ağ ortamlarında karmaşık bağlantılar oldukça sık kopma yaşanabilir, bu nedenle kalp atışı aralığının 1 dakikadan az olması önerilir.

Dış ağ ortamında bağlantının kolayca kopabileceği için bağlantının yeniden kurulabilmesi, uzun süreli bağlantı uygulamaları için gereklidir (bağlantının yeniden kurulması sadece istemci tarafından yapılabilir, sunucu tarafından gerçekleştirilemez). Örneğin, tarayıcı WebSocket, onclose olayını dinlemeli ve bu olay gerçekleştiğinde yeni bir bağlantı kurmalıdır (çökme durumunda bağlantının ertelenmesi gerekir). Daha da önemlisi, sunucu da düzenli aralıklarla kalp atışı verisi göndermeli ve istemci, sunucunun kalp atışı verisinin aşırı zaman aşıp aşılmadığını düzenli aralıklarla kontrol etmeli ve belirli bir süre içinde sunucunun kalp atışı verisi almadığını tespit ederse bağlantıyı kapatmalı ve yeni bir bağlantı kurmalıdır.

