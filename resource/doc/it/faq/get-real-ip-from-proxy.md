## Come ottenere l'IP reale del client attraverso il proxy nginx/apache?

Quando si utilizza nginx/apache come proxy per workerman, in realtà nginx/apache funge da client per workerman, quindi l'IP del client ottenuto su workerman è l'IP del server nginx/apache, non l'IP reale del client. Ecco come ottenere l'IP reale del client:

**Principio:**

nginx/apache passa l'IP reale del client tramite l'intestazione HTTP, ad esempio nella configurazione nginx aggiungere `proxy_set_header X-Real-IP $remote_addr;` all'interno della direttiva `location`. Workerman legge questo valore dall'intestazione e lo salva nell'oggetto `$connection` (GatewayWorker può salvarlo nella variabile `$_SESSION`), quindi può essere letto direttamente quando necessario.

**Nota:**

Questa configurazione è adatta per i protocolli http/https ws/wss. Per ottenere l'IP del client in altri protocolli, è necessario che il server proxy inserisca un blocco di dati IP per passare l'IP reale del client.

**Configurazione nginx simile a quanto segue:**
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
    # Questa parte passa l'IP reale del client tramite l'intestazione HTTP
    proxy_set_header X-Real-IP $remote_addr;
  }
  
  # location / {} Altre configurazioni del sito...
}
```

**Lettura dell'IP del client impostato da nginx da workerman:**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:7272');

// Quando il client si connette e completa la handshaking WebSocket
$worker->onConnect = function(TcpConnection $connection) {
   /**
    * Callback onWebSocketConnect durante la handshaking del client WebSocket
    * Ottiene il valore X_REAL_IP tramite l'intestazione http in onWebSocketConnect
    */
   $connection->onWebSocketConnect = function(TcpConnection $connection){
       /**
        * L'oggetto connection non ha l'attributo realIP, quindi qui si aggiunge dinamicamente
        * l'attributo realIP all'oggetto connection
        */
       $connection->realIP = $_SERVER['HTTP_X_REAL_IP'];
   };
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Quando si utilizza l'IP reale del client, si può usare direttamente $connection->realIP
    $connection->send($connection->realIP);
};
Worker::runAll();
```

**Ottenere l'IP del client impostato da nginx da GatewayWorker:**

Aggiungi il seguente codice a Events.php:

```php
class Events
{
   public static function onWebsocketConnect($client_id, $data)
   {    
        $_SESSION['realIP'] = $data['server']['HTTP_X_REAL_IP'];
   }
   // ... Altri codici omessi...
}
```

Dopo aver aggiunto il codice, è necessario riavviare GatewayWorker.

In questo modo, è possibile ottenere l'IP reale del client in `onMessage` e `onClose` metodi di Events.php utilizzando `$_SESSION['realIP']`.

> Nota: `onWorkerStart`, `onConnect`, `onWorkerStop` in Events.php non possono utilizzare direttamente `$_SESSION['realIP']`.
