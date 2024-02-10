## Nginx/apache üzerinden müşteri gerçek IP'sini nasıl alırım?

Workerman için nginx/apache'yi proxy olarak kullandığınızda, nginx/apache aslında workerman'ın istemcisi gibi hareket eder. Bu durumda workerman üzerinden alınan istemci IP'si, gerçek istemci IP'si değil, nginx/apache sunucusunun IPsi olacaktır. Gerçek istemci IP'sini nasıl alabileceğinizi aşağıdaki yöntemleri kullanarak öğrenebilirsiniz.

**Prensip:**

Nginx/apache, gerçek istemci IP'sini HTTP başlığı vasıtasıyla iletebilir. Örneğin, nginx konfigürasyonundaki location bloğuna ```proxy_set_header X-Real-IP $remote_addr;``` ayarını eklemek. Workerman bu başlık değerini okuyarak, bu değeri ```$connection``` nesnesine kaydeder (GatewayWorker'da ```$_SESSION``` değişkenine kaydedebilirsiniz) ve kullanırken doğrudan değişkeni okuyabilirsiniz.

**Not:**

Aşağıdaki konfigürasyonlar HTTP/HTTPS ve WS/WSS protokolleri için geçerlidir. Diğer protokoller için benzer şekilde istemci IP'sini almaları için proxy sunucularının gerçek istemci IP'sini veri paketine eklemeleri gerekmektedir.

**Nginx konfigürasyonu aşağıdaki gibi olabilir**:
```nginx
server {
  listen 443;

  ssl on;
  ssl_certificate /etc/ssl/server.pem;
  ssl_certificate_key /etc/ssl/server.key;
  ssl_session_timeout 5m;
  ssl_session_cache shared:SSL:50m;
  ssl_protocols SSLv3 SSLv2 TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;

  location /wss
  {
    proxy_pass http://127.0.0.1:8282;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
    # Gerçek istemci IP'sini HTTP başlığı ile aktarmak
    proxy_set_header X-Real-IP $remote_addr;
  }
  
  # location / {} sitesinin diğer konfigürasyonları...
}
```

**Workerman'da nginx tarafından ayarlanan başlıklardan istemci IP'sini okuma:**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:7272');

// İstemci bağlandığında, yani TCP üçlü el sıkışmadan sonra geri çağırılır
$worker->onConnect = function(TcpConnection $connection) {
   /**
    * İstemci websocket el sıkışması sırasında onWebSocketConnect geri çağrı fonksiyonu
    * onWebSocketConnect geri çağrısı içinde, HTTP başlığından X_REAL_IP değerini alırız
    */
   $connection->onWebSocketConnect = function(TcpConnection $connection){
       // connection nesnesi aslında realIP özelliğine sahip değil, bu yüzden burada connection nesnesine dinamik olarak realIP özelliğini ekliyoruz
       $connection->realIP = $_SERVER['HTTP_X_REAL_IP'];
   };
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Gerçek istemci IP'sini kullanırken, doğrudan $connection->realIP değişkenini kullanabilirsiniz
    $connection->send($connection->realIP);
};
Worker::runAll();
```

**GatewayWorker'da nginx tarafından ayarlanan başlıklardan istemci IP'sini almak**

Events.php'ye aşağıdaki kodları ekleyin
```php
class Events
{
   public static function onWebsocketConnect($client_id, $data)
   {    
        $_SESSION['realIP'] = $data['server']['HTTP_X_REAL_IP'];
   }
   // .... Diğer kodlar kısmını atla....
}
```
Kodu ekledikten sonra GatewayWorker'ı yeniden başlatmanız gerekecektir.

Bu şekilde, Events.php'deki `onMessage` ve `onClose` metodları üzerinden `$_SESSION['realIP']` değişkenini kullanarak istemci gerçek IP'sine erişebilirsiniz.

> Not: Events.php'deki `onWorkerStart`, `onConnect`, `onWorkerStop` doğrudan `$_SESSION['realIP']` değişkenini kullanamaz.
