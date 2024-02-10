# Basit Geliştirme Örneği

## Kurulum

**Workerman Kurulumu**
Boş bir dizinde çalıştırın
`composer require workerman/workerman`

## Örnek 1: Dışarıdan Web Hizmeti Sunmak için HTTP Protokolünün Kullanılması
**start.php Dosyası Oluşturma**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

// 2345 portunu dinleyen, http protokolünü kullanan bir Worker oluşturulur
$http_worker = new Worker("http://0.0.0.0:2345");

// 4 adet süreci başlatarak hizmet sunulur
$http_worker->count = 4;

// Tarayıcıdan gelen veriyi alınca tarayıcıya "hello world" yanıtı verilir
$http_worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // Tarayıcıya "hello world" mesajı gönderilir
    $connection->send('hello world');
};

// Worker çalıştırılır
Worker::runAll();
```

**Komut Satırında Çalıştırma (Windows kullanıcıları için [cmd komut satırı](https://baike.baidu.com/item/%E5%91%BD%E4%BB%A4%E6%8F%90%E7%A4%BA%E7%AC%A6?fromtitle=CMD&fromid=1193011&type=syn), buna benzer şekilde)**
```shell
php start.php start
```

**Test**

Sunucu IP'si 127.0.0.1 olarak varsayalım

Tarayıcıda şu URL'yi ziyaret edin: http://127.0.0.1:2345

**Not:** 

1. Eğer erişim sorunuyla karşılaşılırsa, [İstemci Bağlantı Hatası Nedenleri](../faq/client-connect-fail.md) başlığına bakın.
2. Sunucu http protokolü kullanır, sadece http protokolüyle iletişim kurulabilir, websocket gibi diğer protokollerle doğrudan iletişim kurulamaz.

## Örnek 2: Dışarıdan Servis Sağlamak için WebSocket Protokolünün Kullanılması
**ws_test.php Dosyası Oluşturma**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Not: Burada önceki örnekle farklı olarak websocket protokolü kullanılır
$ws_worker = new Worker("websocket://0.0.0.0:2000");

// 4 adet süreci başlatarak hizmet sunulur
$ws_worker->count = 4;

// İstemciden veri alındığında "hello $data" yanıtı verilir
$ws_worker->onMessage = function(TcpConnection $connection, $data)
{
    // İstemciye "hello $data" mesajı gönderilir
    $connection->send('hello ' . $data);
};

// Worker çalıştırılır
Worker::runAll();
```

**Komut Satırında Çalıştırma**
```shell
php ws_test.php start
```

**Test**

Chrome tarayıcısını açın, F12'ye basarak hata ayıklama konsolunu açın. Konsolda aşağıdaki kodu girin (veya aşağıdaki kodu html sayfasına yerleştirin ve javascript kullanarak çalıştırın).

```javascript
// Sunucu IP'si 127.0.0.1 olarak varsayalım
ws = new WebSocket("ws://127.0.0.1:2000");
ws.onopen = function() {
    alert("Bağlantı başarılı");
    ws.send('tom');
    alert("Sunucuya bir dize gönderildi: tom");
};
ws.onmessage = function(e) {
    alert("Sunucudan gelen mesaj: " + e.data);
};
```

**Not:** 

1. Eğer erişim sorunuyla karşılaşılırsa, [Manuel Yaygın Sorunlar-Bağlantı Başarısız](../faq/client-connect-fail.md) başlığına bakın.
2. Sunucu websocket protokolünü kullanır, sadece websocket protokolüyle iletişim kurulabilir, http gibi diğer protokollerle doğrudan iletişim kurulamaz.

## Örnek 3: Veri Transferi İçin Doğrudan TCP Kullanımı
**tcp_test.php Dosyası Oluşturma**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Herhangi bir uygulama katmanı protokolü kullanmayan 2347 portunu dinleyen Worker oluşturulur
$tcp_worker = new Worker("tcp://0.0.0.0:2347");

// 4 adet süreci başlatarak hizmet sunulur
$tcp_worker->count = 4;

// İstemciden veri alındığında "hello $data" yanıtı verilir
$tcp_worker->onMessage = function(TcpConnection $connection, $data)
{
    // İstemciye "hello $data" mesajı gönderilir
    $connection->send('hello ' . $data);
};

// Worker çalıştırılır
Worker::runAll();
```

**Komut Satırında Çalıştırma**

```shell
php tcp_test.php start
```

**Test: Komut Satırında Çalıştırma**
(Aşağıdaki linux komut satırı örneğidir, windows'ta farklı bir şekilde çalışacaktır)
```shell
telnet 127.0.0.1 2347
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
tom
hello tom
```

**Not:**

1. Eğer erişim sorunuyla karşılaşılırsa, [Manuel Yaygın Sorunlar-Bağlantı Başarısız](../faq/client-connect-fail.md) başlığına bakın.
2. Sunucu saf TCP protokolünü kullanır, websocket, http gibi diğer protokollerle doğrudan iletişim kurulamaz.
