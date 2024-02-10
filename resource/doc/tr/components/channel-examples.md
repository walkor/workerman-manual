- **``` (Workerman versiyonu >=3.3.0 gerektirir) ```**

Worker tabanlı çoklu süreç (dağıtık küme) itme sistem, küme itme, küme yayını.

`start_channel.php` 
Tüm sistem sadece bir tane start_channel servisi dağıtılabilir. 192.168.1.1'de çalıştığı varsayılmaktadır.
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Bir Kanal sunucusu başlat
$channel_server = new Channel\Server('0.0.0.0', 2206);
Worker::runAll();
```

`start_ws.php`
Tüm sistem 192.168.1.2 ve 192.168.1.3'teki iki sunucuda çalışabilir start_ws hizmetleri, çalışma.
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// websocket sunucusu
$worker = new Worker('websocket://0.0.0.0:4236');
$worker->count=2;
$worker->name = 'pusher';
$worker->onWorkerStart = function($worker)
{
    // Kanal istemci, Kanal sunucusuna bağlan
    Channel\Client::connect('192.168.1.1', 2206);
    // Kendi işlem kimliği ile bir etkinlik adı
    $event_name = $worker->id;
    // worker->id etkinliğine abone ol ve etkinlik işleme fonksiyonunu kaydet
    Channel\Client::on($event_name, function($event_data)use($worker){
        $to_connection_id = $event_data['to_connection_id'];
        $message = $event_data['content'];
        if(!isset($worker->connections[$to_connection_id]))
        {
            echo "bağlantı mevcut değil\n";
            return;
        }
        $to_connection = $worker->connections[$to_connection_id];
        $to_connection->send($message);
    });

    // Yayın etkinliğine abone ol
    $event_name = 'yayın';
    // Yayın etkinliğini aldığında olay verisini geçerli süreç içindeki tüm istemci bağlantılarına gönderir
    Channel\Client::on($event_name, function($event_data)use($worker){
        $message = $event_data['content'];
        foreach($worker->connections as $connection)
        {
            $connection->send($message);
        }
    });
};

$worker->onConnect = function(TcpConnection $connection)use($worker)
{
    $msg = "workerID:{$worker->id} connectionID:{$connection->id} bağlandı\n";
    echo $msg;
    $connection->send($msg);
};
Worker::runAll();
```

`start_http.php` 
Tüm sistem 192.168.1.4 ve 192.168.1.5'teki iki sunucuda çalışabilir start_ws hizmetleri, çalışma.
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Herhangi bir istemciye veri iten bir http isteği işlemek için kullanılır, workerID ve connectionID geçirmek zorunludur
$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->name = 'yayımcı';
$http_worker->onWorkerStart = function()
{
    Channel\Client::connect('192.168.1.1', 2206);
};
$http_worker->onMessage = function(TcpConnection $connection, $request)
{
    // workerman 4.x uyumluluğu
    if (!is_array($request)) {
            $_GET = $request->get();
   }
    $connection->send('tamam');
    if(empty($_GET['content'])) return;
    // Belirli bir işlemci sürecindeki belirli bir bağlantıya veri itmek
    if(isset($_GET['to_worker_id']) && isset($_GET['to_connection_id']))
    {
        $event_name = $_GET['to_worker_id'];
        $to_connection_id = $_GET['to_connection_id'];
        $content = $_GET['content'];
        Channel\Client::publish($event_name, array(
           'to_connection_id' => $to_connection_id,
           'content'          => $content
        ));
    }
    // Genel yayın verisi
    else
    {
        $event_name = 'yayın';
        $content = $_GET['content'];
        Channel\Client::publish($event_name, array(
           'content'          => $content
        ));
    }
};
Worker::runAll();
```

## Test 
1. Her sunucuda hizmetleri başlatın.
2. İstemci, sunucuya bağlanın.
   Chrome tarayıcısını açın, F12'ye basarak hata ayıklama konsolunu açın, Console sekmesine gidin ve aşağıdaki komutları girin (veya aşağıdaki kodları bir HTML sayfasına yerleştirin ve çalıştırın):
```javascript
// ayrıca ws://192.168.1.3:4236' da bağlanabilirsiniz
ws = new WebSocket("ws://192.168.1.2:4236");
ws.onmessage = function(e) {
    alert("Sunucudan gelen mesaj: " + e.data);
};
```
3. HTTP arayüzünü kullanarak veri itme
```http://192.168.1.4:4237/?content={$content}```  veya  ```http://192.168.1.5:4237/?content={$content}```  adresine giderek tüm istemci bağlantılarına ```$content``` verisini itin.
```http://192.168.1.4:4237/?to_worker_id={$worker_id}&to_connection_id={$connection_id}&content={$content}``` ya da ```http://192.168.1.5:4237/?to_worker_id={$worker_id}&to_connection_id={$connection_id}&content={$content}```  adresine giderek belirli bir işlemci sürecindeki belirli bir istemci bağlantısına ```$content``` verisini itin.

Not: Test sırasında  ```{$worker_id}```   ```{$connection_id}``` ve ```{$content}```  değerlerini gerçek değerlerle değiştirin.
