## ¿Cómo obtener la IP real del cliente a través de un proxy nginx/apache?

Al utilizar nginx/apache como proxy para workerman, nginx/apache actúa en realidad como el cliente de workerman, por lo que la IP del cliente obtenida en workerman es la IP del servidor nginx/apache, no la IP real del cliente. A continuación se describe cómo obtener la IP real del cliente.

**Principio:**

nginx/apache pasa la IP real del cliente a través de la cabecera HTTP, por ejemplo, agregando ```proxy_set_header X-Real-IP $remote_addr;``` en la configuración de location en nginx. Workerman lee este valor de la cabecera para guardarlo en el objeto ```$connection``` (en GatewayWorker se puede guardar en la variable ```$_SESSION```), y se puede acceder a él directamente cuando se necesite.

**Nota:**

La siguiente configuración es aplicable a los protocolos http/https ws/wss. Para obtener la IP del cliente en otros protocolos, se necesita que el servidor proxy inserte una sección de datos IP para pasar la IP real del cliente.

**Configuración similar en nginx:**
``` 
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
    # Esta parte utiliza la cabecera HTTP para pasar la IP real del cliente
    proxy_set_header X-Real-IP $remote_addr;
  }
  
  # location / {} - Otras configuraciones del sitio...
}
```

**Obtener la IP del cliente desde la cabecera configurada en nginx:**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:7272');

$worker->onConnect = function(TcpConnection $connection) {
   $connection->onWebSocketConnect = function(TcpConnection $connection){
       $connection->realIP = $_SERVER['HTTP_X_REAL_IP'];
   };
};

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send($connection->realIP);
};

Worker::runAll();
```

**Obtener la IP del cliente desde la cabecera configurada en nginx en GatewayWorker:**

Agrega el siguiente código en Events.php
```php
class Events
{
   public static function onWebsocketConnect($client_id, $data)
   {    
        $_SESSION['realIP'] = $data['server']['HTTP_X_REAL_IP'];
   }
   // Resto del código omitido...
}
```
Después de agregar este código, es necesario reiniciar GatewayWorker.

De esta forma, se puede obtener la IP real del cliente en los métodos `onMessage` y `onClose` en Events.php a través de `$_SESSION['realIP']`.

> Nota: Los métodos `onWorkerStart`, `onConnect`, `onWorkerStop` en Events.php no pueden utilizar directamente `$_SESSION['realIP']`.
