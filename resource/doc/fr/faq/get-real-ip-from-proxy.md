## Comment obtenir l'adresse IP réelle du client à travers un proxy nginx/apache ?

Lorsque vous utilisez nginx/apache comme proxy pour workerman, nginx/apache agit en réalité comme le client de workerman. Par conséquent, l'adresse IP du client obtenue dans workerman est celle du serveur nginx/apache et non l'adresse IP réelle du client. Pour obtenir l'adresse IP réelle du client, vous pouvez suivre la méthode ci-dessous.

**Principe :**

nginx/apache transmet l'adresse IP réelle du client via l'en-tête http, par exemple en ajoutant ```proxy_set_header X-Real-IP $remote_addr;``` dans la configuration nginx. Workerman lit ensuite cette valeur d'en-tête et la stocke dans l'objet ```$connection``` (GatewayWorker peut la stocker dans la variable ```$_SESSION```), que vous pouvez simplement récupérer lorsque nécessaire.

**Remarque :**

Cette configuration est applicable aux protocoles http/https et ws/wss. Pour d'autres protocoles, la méthode pour obtenir l'adresse IP réelle du client est similaire, mais nécessite que le serveur proxy insère une section de données IP pour transmettre l'adresse IP réelle du client.

**Exemple de configuration nginx :**

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
    # Utilisation de l'en-tête http pour transmettre l'adresse IP réelle du client
    proxy_set_header X-Real-IP $remote_addr;
  }
  
  # Configuration supplémentaire du site...
}
```

**Extraction de l'adresse IP réelle du client à partir de l'en-tête configuré dans workerman :**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:7272');

// Callback lorsque le client se connecte après l'établissement du handshake TCP
$worker->onConnect = function(TcpConnection $connection) {
   /**
    * Callback onWebSocketConnect lors de la négociation du handshake websocket
    * Récupération de la valeur X_REAL_IP à partir de l'en-tête http de nginx dans onWebSocketConnect
    */
   $connection->onWebSocketConnect = function(TcpConnection $connection){
       /**
        * L'objet connection n'a pas réellement la propriété realIP, nous lui ajoutons dynamiquement
        * Souvenez-vous que les objets PHP peuvent avoir des propriétés ajoutées dynamiquement, vous pouvez utiliser le nom de propriété que vous préférez
        */
       $connection->realIP = $_SERVER['HTTP_X_REAL_IP'];
   };
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Lors de l'utilisation de l'adresse IP réelle du client, utilisez simplement $connection->realIP
    $connection->send($connection->realIP);
};
Worker::runAll();
```

**Obtention de l'adresse IP réelle du client à partir de l'en-tête configuré dans GatewayWorker :**

Ajoutez le code suivant dans Events.php
```php
class Events
{
   public static function onWebsocketConnect($client_id, $data)
   {    
        $_SESSION['realIP'] = $data['server']['HTTP_X_REAL_IP'];
   }
   // .... Autres codes omis....
}
```
Après avoir ajouté ce code, redémarrez GatewayWorker.

De cette manière, vous pouvez obtenir l'adresse IP réelle du client dans les méthodes `onMessage` et `onClose` de Events.php en utilisant `$_SESSION['realIP']`.

> Remarque : Les méthodes `onWorkerStart`, `onConnect` et `onWorkerStop` dans Events.php ne peuvent pas utiliser directement `$_SESSION['realIP']`.
