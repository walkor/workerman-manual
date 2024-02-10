## How to get the actual client IP address through nginx/apache proxy?

When using nginx/apache as a proxy for Workerman, nginx/apache actually acts as the client for Workerman. As a result, the IP address obtained in Workerman will be that of the nginx/apache server, not the actual client's IP address. The following method can be used to obtain the actual client IP address.


**Principle:**

nginx/apache passes the client's actual IP address through the HTTP header, for example, by setting the following in the nginx configuration within the location block:
``` 
proxy_set_header X-Real-IP $remote_addr;
```
Workerman reads this header value and saves it to the `$connection object` (or `$_SESSION` variable in GatewayWorker). The IP address can be directly accessed when needed.


**Note:**

The following configuration is suitable for http/https ws/wss protocols. Similar methods are required for obtaining the client IP address for other protocols, where the proxy server needs to insert a segment of IP data to pass the actual client IP address.


**Sample nginx configuration:**
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
    # This part is used to pass the actual client IP through the HTTP header
    proxy_set_header X-Real-IP $remote_addr;
  }
  
  # location / {} Other configurations for the site...
}
```


**Obtaining the client IP from the header set by nginx in Workerman:**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:7272');

// Callback when a client connects, i.e., after the TCP three-way handshake
$worker->onConnect = function(TcpConnection $connection) {
   /**
    * Callback onWebSocketConnect during the client's websocket handshake
    * Access X_REAL_IP value set by nginx through onWebSocketConnect callback
    */
   $connection->onWebSocketConnect = function(TcpConnection $connection){
       /**
        * The connection object does not have a realIP property, so we dynamically add a realIP property to the connection object
        * Remember that PHP objects can have properties added dynamically, you can use your preferred property name
        */
       $connection->realIP = $_SERVER['HTTP_X_REAL_IP'];
   };
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Access the actual client IP address directly using $connection->realIP
    $connection->send($connection->realIP);
};
Worker::runAll();
```

**Obtaining the client IP from the header set by nginx in GatewayWorker:**

Add the following code to Events.php:
```php
class Events
{
   public static function onWebsocketConnect($client_id, $data)
   {    
        $_SESSION['realIP'] = $data['server']['HTTP_X_REAL_IP'];
   }
   // .... Other code omitted....
}
```
After adding the code, restart GatewayWorker.

By using `$_SESSION['realIP']` in the `onMessage` and `onClose` methods in Events.php, the actual client IP address can be obtained.

> Note: `$_SESSION['realIP']` cannot be directly used in `onWorkerStart`, `onConnect`, and `onWorkerStop` in Events.php.
