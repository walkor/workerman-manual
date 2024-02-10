# ws protokolü

Şu anda Workerman'ın **ws protokolü sürümü 13**'tür.

Workerman, ws protokolü aracılığıyla bir websocket bağlantısı başlatarak uzak websocket sunucusuna client olarak kullanılabilir ve çift yönlü iletişim sağlayabilir.

> **Not**
> ws protokolü sadece bir AsyncTcpConnection olarak client olarak kullanılabilir ve websocket server olarak dinleme protokolü olarak kullanılamaz. Yani aşağıdaki yazım yanlıştır.

```php
$worker = new Worker('ws://0.0.0.0:8080');
```

Eğer Workerman'ı websocket server olarak kullanmak istiyorsanız, lütfen [websocket protokolünü](about-websocket.md) kullanın.

**ws olarak websocket client protokolü örneği:**

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Protocols\Ws;
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;

$worker = new Worker();
// Process starts
$worker->onWorkerStart = function()
{
    // Uzak websocket sunucusuna websocket protokolü ile bağlan
    $ws_connection = new AsyncTcpConnection("ws://127.0.0.1:1234");
    // Sunucuya her 55 saniyede bir 0x9 opcode'lu bir websocket kalp atışı gönder (isteğe bağlı)
    $ws_connection->websocketPingInterval = 55;
    // HTTP header'ları ayarla (isteğe bağlı)
    $ws_connection->headers = [
        'Cookie' => 'PHPSID=82u98fjhakfusuanfnahfi; token=2hf9a929jhfihaf9i',
        'OtherKey' => 'values'
    ];
    // Veri türünü ayarla (isteğe bağlı)
    $ws_connection->websocketType = Ws::BINARY_TYPE_BLOB; // BINARY_TYPE_BLOB: metin, BINARY_TYPE_ARRAYBUFFER: ikili
    // Üç el sıkışma tamamlandıktan sonra (isteğe bağlı)
    $ws_connection->onConnect = function($connection){
        echo "tcp connected\n";
    };
    // Websocket el sıkışması tamamlandıktan sonra (isteğe bağlı)
    $ws_connection->onWebSocketConnect = function(AsyncTcpConnection $con, $response) {
        echo $response;
        $con->send('hello');
    };
    // Uzak websocket sunucusundan mesaj alındığında
    $ws_connection->onMessage = function($connection, $data){
        echo "recv: $data\n";
    };
    // Bağlantıda hata oluştuğunda, genellikle uzak websocket sunucusuna bağlantı hatası (isteğe bağlı) 
    $ws_connection->onError = function($connection, $code, $msg){
        echo "error: $msg\n";
    };
    // Bağlanılan uzak websocket sunucusu bağlantısı kesildiğinde (isteğe bağlı, yeniden bağlantı önerilir)
    $ws_connection->onClose = function($connection){
        echo "connection closed and try to reconnect\n";
        // Bağlantı kesildiğinde, 1 saniye sonra yeniden bağlan
        $connection->reConnect(1);
    };
    // Yukarıdaki geri aramaları ayarladıktan sonra, bağlantı işlemini gerçekleştir
    $ws_connection->connect();
};
Worker::runAll();
```

Daha fazla bilgi için [ws/wss client olarak](../faq/as-wss-client.md) referans alın.
