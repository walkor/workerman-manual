# WebSocket Protokolü

Şu anda Workerman'ın **WebSocket protokolü sürümü 13**'tür.

WebSocket protokolü, HTML5'in yeni bir protokolüdür. Tarayıcı ve sunucu arasında çift yönlü iletişimi gerçekleştirir.

## WebSocket ve TCP İlişkisi

WebSocket ve HTTP gibi bir uygulama katmanı protokolüdür, her ikisi de TCP üzerinden iletişim sağlar, WebSocket'in kendisi ve Soket arasında büyük bir ilişki yoktur ve eş anlamlı değildir.

## WebSocket Protokolü El Sıkışması

WebSocket protokolünde bir el sıkışma süreci vardır, el sıkışması sırasında tarayıcı ve sunucu HTTP protokolü aracılığıyla iletişim kurar, Workerman'da el sıkışma sürecine şu şekilde müdahale edilebilir.

**workerman <= 4.1 olduğunda**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Connection\TcpConnection;
use Workerman\Worker;

$ws = new Worker('websocket://0.0.0.0:8181');
$ws->onConnect = function($connection)
{
    $connection->onWebSocketConnect = function($connection , $httpBuffer)
    {
        // Bağlantı kaynağının uygun olup olmadığını burada kontrol edebilir, uygun değilse bağlantıyı kapat
        // $_SERVER['HTTP_ORIGIN'] bağlantının hangi site sayfasından websocket bağlantısının başlatıldığını belirtir
        if($_SERVER['HTTP_ORIGIN'] != 'https://www.workerman.net')
        {
            $connection->close();
        }
        // onWebSocketConnect içerisinde $_GET $_SERVER kullanılabilir
        // var_dump($_GET, $_SERVER);
    };
};
Worker::runAll();
```

**workerman >= 5.0 olduğunda**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Worker;

$worker = new Worker('websocket://0.0.0.0:12345');
$worker->onWebSocketConnect = function (TcpConnection $connection, Request $request) {
    if ($request->header('origin') != 'https://www.workerman.net') {
        $connection->close();
    }
    var_dump($request->get());
    var_dump($request->header());
};
Worker::runAll();
```

## WebSocket Protokolüyle Binary Veri Aktarımı

WebSocket protokolü varsayılan olarak yalnızca UTF-8 metin iletimine izin verir, binary veri iletimi için aşağıdaki bölümü okuyunuz.

websocket protokolünde protokol başlığında iletilen verinin binary veri mi yoksa UTF-8 metin verisi mi olduğunu belirten bir bayrağı kullanır, tarayıcı bayrağı ve iletilen içerik türünü doğrulayacaktır, eğer uygun değilse bağlantıyı keser.

Bu nedenle sunucu veri gönderirken iletim veri türüne göre bu bayrağı ayarlaması gerekir, Workerman'da normal UTF-8 metin ise bu bayrağı ayarlamak için şunları kullanılır (genellikle bu değer varsayılan olarak ayarlıdır, genellikle manuel olarak ayarlanmasına gerek yoktur)
```php
use Workerman\Protocols\Websocket;
$connection->websocketType = Websocket::BINARY_TYPE_BLOB;
```

Eğer binary veri ise, şunları kullanılır
```php
use Workerman\Protocols\Websocket;
$connection->websocketType = Websocket::BINARY_TYPE_ARRAYBUFFER;
```

**Not**: Eğer $connection->websocketType belirtilmemişse, $connection->websocketType varsayılan olarak BINARY_TYPE_BLOB'dur (yani UTF-8 metin türüdür). Genellikle başvurular UTF-8 metin aktarır, örneğin JSON verisi gönderirken, bu nedenle manuel olarak $connection->websocketType ayarlanmasına gerek yoktur. Yalnızca binary veri aktarılırken (örneğin resim verisi, protobuffer verisi vb.) bu özelliğin BINARY_TYPE_ARRAYBUFFER olarak ayarlanması gerekir.

## Workerman'ı WebSocket İstemcisi Olarak Kullanma

[AsyncTcpConnection sınıfı](../async-tcp-connection.md) ve [ws protokolü](about-ws.md) ile birlikte Workerman'ı uzak WebSocket sunucusuna bağlanan WebSocket istemcisi olarak kullanabilirsiniz, çift yönlü gerçek zamanlı iletişimi tamamlayabilirsiniz.
