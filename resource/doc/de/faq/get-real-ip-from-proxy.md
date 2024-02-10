## Wie kann die tatsächliche IP-Adresse des Clients über einen nginx/apache-Proxy abgerufen werden?
Wenn nginx/apache als Workerman-Proxy verwendet wird, fungieren nginx/apache tatsächlich als Client von Workerman. Daher wird die Client-IP-Adresse, die von Workerman abgerufen wird, die IP-Adresse des nginx/apache-Servers und nicht die tatsächliche IP-Adresse des Clients sein. Nachfolgend finden Sie eine Methode, wie die tatsächliche IP-Adresse des Clients abgerufen werden kann.

**Prinzip:**

nginx/apache leitet die tatsächliche IP-Adresse des Clients über den HTTP-Header weiter, z. B. durch Hinzufügen von ```proxy_set_header X-Real-IP $remote_addr;``` in der nginx-Konfiguration im Location-Bereich. Workerman liest diesen Header-Wert und speichert ihn im ```$connection```-Objekt (GatewayWorker kann ihn in der ```$_SESSION```-Variable speichern). Sie können die Variable dann direkt verwenden, wenn Sie sie benötigen.

**Hinweis:**

Die folgende Konfiguration gilt für die Protokolle http/https sowie ws/wss. Für andere Protokolle ist die Methode zum Abrufen der Client-IP-Adresse ähnlich, sie erfordert jedoch, dass der Proxy-Server die tatsächliche Client-IP-Adresse in ein Datenpaket einfügt, um die IP-Adresse transparent weiterzuleiten.

**Beispiel für eine nginx-Konfiguration:**
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
    # In diesem Abschnitt wird die tatsächliche Client-IP-Adresse über den HTTP-Header transparent weitergeleitet
    proxy_set_header X-Real-IP $remote_addr;
  }
  
  # Standort-URL {} und andere Konfigurationen der Website...
}
```

**Workerman liest die vom nginx gesetzten Header und erhält die Client-IP:**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:7272');

// Wenn ein Client verbunden ist, nach dem Abschluss des TCP-Drei-Wege-Handshakes
$worker->onConnect = function(TcpConnection $connection) {
   /**
    * Callback onWebSocketConnect, wenn ein WebSocket-Handshake vom Client ausgeführt wird,
    * hier erhält man den Wert von X-REAL-IP aus dem Header über `$_SERVER['HTTP_X_REAL_IP']`
    */
   $connection->onWebSocketConnect = function(TcpConnection $connection){
       // Das "realIP"-Attribut wird dynamisch dem Connection-Objekt hinzugefügt (PHP-Objekte können dynamisch Attribute hinzufügen)
       $connection->realIP = $_SERVER['HTTP_X_REAL_IP'];
   };
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Bei Verwendung der tatsächlichen Client-IP einfach `$connection->realIP` verwenden
    $connection->send($connection->realIP);
};
Worker::runAll();
```

**GatewayWorker holt die Client-IP aus dem von nginx gesetzten Header:**

Fügen Sie folgenden Code zu `Events.php` hinzu:
```php
class Events
{
   public static function onWebsocketConnect($client_id, $data)
   {    
        $_SESSION['realIP'] = $data['server']['HTTP_X_REAL_IP'];
   }
   // .... Anderer Code wird hier ausgelassen ....
}
```
Nach dem Hinzufügen des Codes müssen Sie GatewayWorker neu starten.

Auf diese Weise können Sie in den `onMessage`- und `onClose`-Methoden von `Events.php` die tatsächliche Client-IP-Adresse über `$_SESSION['realIP']` erhalten.

> Hinweis: Direkter Zugriff auf `$_SESSION['realIP']` ist in `onWorkerStart`, `onConnect` und `onWorkerStop` in `Events.php` nicht möglich.
