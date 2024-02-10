# webman olarak ws/wss istemcisi olarak

Bazen workerman'ı ws/wss protokolü kullanarak bir sunucuya bağlanmak ve etkileşimde bulunmak gerekebilir. Aşağıda örnekler bulunmaktadır.

## webman olarak ws istemcisi

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){

    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    
    // websocket el sıkışması başarılı olduğunda
    $con->onWebSocketConnect = function(AsyncTcpConnection $con, ) {
        $con->send('hello');
    };

    // Mesaj alındığında
    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## webman olarak wss(ws+ssl) istemcisi

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // ssl için 443 portuna erişilmesi gerekmektedir
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443');

    // SSL şifreleme yöntemiyle erişim sağlamak için wss'ye dönüştürülür
    $con->transport = 'ssl';

    $con->onWebSocketConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## webman olarak wss(ws+ssl) istemcisi+yerel SSL sertifikası

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // Karşıdaki ana bilgisayarın yerel IP ve bağlantı noktası ile ssl sertifikası ayarlanır
    $context_option = array(
        // ssl seçenekleri, bkz. http://php.net/manual/context.ssl.php
        'ssl' => array(
            // Yerel sertifika yolu. PEM formatında olmalı ve yerel sertifika ve özel anahtar içermelidir.
            'local_cert'        => '/your/path/to/pemfile',
            // local_cert dosyasının şifresi.
            'passphrase'        => 'your_pem_passphrase',
            // Kendi imzalı sertifikalara izin verilsin mi?
            'allow_self_signed' => true,
            // SSL sertifikasının doğrulanması gerekiyor mu?
            'verify_peer'       => false
        )
    );

    // ssl için 443 portuna erişilmesi gerekmektedir
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443', $context_option);

    // SSL ile şifrelenmiş erişim sağlamak için wss'ye dönüştürülür
    $con->transport = 'ssl';

    $con->onWebSocketConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## Diğer Ayarlar
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Protocols\Ws;
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;

$worker = new Worker();
// İşçi başlatıldığında
$worker->onWorkerStart = function()
{
    // Uzak bir websocket sunucusuna websocket protokolüyle bağlan
    $ws_connection = new AsyncTcpConnection("ws://127.0.0.1:1234");
    // Her 55 saniyede bir opcode değeri 0x9 olan bir websocket kalp atışı gönder
    $ws_connection->websocketPingInterval = 55;
    // Özel http başlıkları
    $ws_connection->headers = ['token' => 'value'];
    // Veri türünü ayarla, varsayılan BINARY_TYPE_BLOB metindir
    $ws_connection->websocketType = Ws::BINARY_TYPE_BLOB; // BINARY_TYPE_BLOB metin BINARY_TYPE_ARRAYBUFFER ikili
    // TCP üç el sıkışma tamamlandığında
    $ws_connection->onConnect = function($connection){
        echo "tcp connected\n";
    };
    // Websocket el sıkışması tamamlandığında 
    $ws_connection->onWebSocketConnect = function(AsyncTcpConnection $con, $response) {
        echo $response;
        $con->send('hello');
    };
    // Uzak websocket sunucusu bir mesaj gönderdiğinde
    $ws_connection->onMessage = function($connection, $data){
        echo "alınan: $data\n";
    };
    // Bağlantıda bir hata oluştuğunda, genellikle bağlantı uzak websocket sunucusuna başarısız olduğu hata
    $ws_connection->onError = function($connection, $code, $msg){
        echo "hata: $msg\n";
    };
    // Uzak websocket sunucusuna bağlantı koparsa
    $ws_connection->onClose = function($connection){
        echo "bağlantı kapatıldı ve tekrar bağlanmaya çalışılıyor\n";
        // Bağlantı koptuysa, 1 saniye sonra yeniden bağlan
        $connection->reConnect(1);
    };
    // Yukarıdaki tüm geri aramaları ayarladıktan sonra, bağlantı işlemini gerçekleştir
    $ws_connection->connect();
};
Worker::runAll();
```
