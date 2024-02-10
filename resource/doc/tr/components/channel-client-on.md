```php
# on
**``` (Workerman sürümü >=3.3.0 gerektirir) ```**
```php
void \Channel\Client::on(string $event_name, callback $callback_function)
```
```$event_name``` olayını abone olur ve olay gerçekleştiğinde ```$callback_function``` geri çağrıyı kaydeder

## Geri çağrı fonksiyonunun parametreleri

 ``` $event_name ```
Abone olunan olayın adı, herhangi bir dize olabilir.

 ``` $callback_function ```
Olay gerçekleştiğinde tetiklenen geri çağrı fonksiyonu. Fonksiyon prototipi ```callback_function(mixed $event_data)``` şeklindedir. ```$event_data```, olay yayımlandığında iletilen olay verileridir.

Not: 
Aynı olay için iki geri çağrı fonksiyonu kaydedilirse, sonraki geri çağrı fonksiyonu öncekinden üstün olacaktır.

## Örnek
Çoklu işlemli Worker (multi-server), bir istemci bir mesaj gönderirse, tüm istemcilere yayınla

start.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Bir Channel sunucusunu başlat
$channel_server = new Channel\Server('0.0.0.0', 2206);

// Websocket sunucusu
$worker = new Worker('websocket://0.0.0.0:4236');
$worker->name = 'websocket';
$worker->count = 6;
// Her worker işlemi başlatıldığında
$worker->onWorkerStart = function($worker)
{
    // Channel istemcisi, Channel sunucusuna bağlanır
    Channel\Client::connect('127.0.0.1', 2206);
    // yayın olayını abone olur ve olay gerçekleştiğinde tetiklenen geri çağrıyı kaydeder
    Channel\Client::on('broadcast', function($event_data)use($worker){
        // Mevcut worker işlemine bağlı tüm istemcilere mesajı yayınla
        foreach($worker->connections as $connection)
        {
            $connection->send($event_data);
        }
    });
};

$worker->onMessage = function(TcpConnection $connection, $data)
{
   // İstemci tarafından gelen verileri olay verisi olarak kullan
   $event_data = $data;
   // Tüm worker işlemlerine yayın olayını yayınla
   \Channel\Client::publish('broadcast', $event_data);
};

Worker::runAll();
```

**Test**

Chrome tarayıcıyı açın, F12 tuşuna basarak hata ayıklama konsolunu açın, Console sekmesinde aşağıdaki kodları girin (veya aşağıdaki kodları html sayfasına yerleştirip js ile çalıştırın)

Mesajı alan bağlantı
```javascript
// 127.0.0.1, gerçek Workerman'ın bulunduğu ip ile değiştirilmelidir
ws = new WebSocket("ws://127.0.0.1:4236");
ws.onmessage = function(e) {
    alert("Sunucudan gelen mesaj: " + e.data);
};
```

Mesajı yayınla
```javascript
ws.send('hello world');
```
