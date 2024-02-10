# Webman Dökümantasyonu

## Giriş
Webman, yüksek performanslı bir PHP çerçevesine dayanan Workerman framework'üdür. Webman, HTTP, WebSocket ve diğer ağ protokolleri için olay tabanlı ağ uygulamaları geliştirmek için kullanılır. Bu belgede Webman'in kullanımı, özellikleri ve yapılandırması hakkında bilgi bulabilirsiniz.

## Yapılandırma
Webman, bir HTTP veya WebSocket sunucusunu yapılandırmak için kullanılabilir. Her iki türde de benzer bir yapılandırma süreci vardır. Sunucu yapılandırmasını gerçekleştirmek için bir yapılandırma dosyası oluşturmanız ve gerekli ayarları belirtmeniz gerekir.

Örnek HTTP Sunucusu Yapılandırması:

```php
use Webman\Http;

$server = new Http\Server('http://0.0.0.0:2345');

$server->on('request', function ($connection, $request) {
    $connection->send('Hello, World!');
});

$server->start();
```

Örnek WebSocket Sunucusu Yapılandırması:

```php
use Webman\WebSocket;

$server = new WebSocket\Server('websocket://0.0.0.0:1234');

$server->on('connect', function ($connection) {
    $connection->send('Hello, WebSocket!');
});

$server->start();
```

## İstemci Bağlantısı
Webman, istemci bağlantılarını kolayca yönetmenizi sağlar. İstemci bağlantılarından gelen verileri işleyebilir ve istemci bağlantılarına veri gönderebilirsiniz. Örneğin, HTTP sunucusuna gelen istekleri işleyebilir ve istemcilere yanıt olarak veri gönderebilirsiniz.

```php
$server->on('request', function ($connection, $request) {
    // Istemci isteğini işle
    $connection->send('Hello, World!');
});
```

## Etkinlikler ve Geri çağrılar
Webman, çeşitli ağ etkinlikleri için olaylar ve geri çağrılar sağlar. Örneğin, istemci bağlantılarından gelen verileri işlemek için "onMessage" olayını kullanabilirsiniz.

```php
$server->on('message', function ($connection, $data) {
    // İstemci verisini işle
});
```

Bu örneklerde sunucu ve istemci tarafıyla ilgili temel bilgileri bulabilirsiniz. Webman'in diğer özellikleri ve ayrıntıları hakkında daha fazla bilgi için resmi dökümantasyonu inceleyebilirsiniz.
