# Create wss Service

**Question:**

How can Workerman create a wss service, so that clients can connect to communicate using the wss protocol, for example, to connect to the server in WeChat Mini Program?

**Answer:**

In fact, the wss protocol is a combination of [websocket](https://en.wikipedia.org/wiki/WebSocket) and [SSL](https://en.wikipedia.org/wiki/Transport_Layer_Security), which is the addition of an [SSL](https://en.wikipedia.org/wiki/Transport_Layer_Security) layer on top of the websocket protocol, similar to [https](https://en.wikipedia.org/wiki/HTTPS) ([http](https://en.wikipedia.org/wiki/HTTP) + [SSL](https://en.wikipedia.org/wiki/Transport_Layer_Security)).
Therefore, to support the wss protocol, it is only necessary to enable [SSL](https://en.wikipedia.org/wiki/Transport_Layer_Security) on top of the websocket protocol.

## Method 1: Use nginx/apache to Proxy SSL (Recommended)

**Communication Principle and Process:**

1. The client initiates a wss connection to nginx/apache.
2. Nginx/apache converts the wss protocol data into ws protocol data and forwards it to Workerman's websocket protocol port.
3. Workerman handles the received data and performs business logic processing.
4. When Workerman sends a message to the client, the data is converted to the wss protocol by nginx/apache and then sent to the client.

## Nginx Configuration Reference

**Prerequisites and Preparation:**

1. Nginx is installed, version not below 1.3.
2. Workerman is listening on port 8282 (websocket protocol).
3. SSL certificate (pem/crt file and key file) has been obtained and is placed in /etc/nginx/conf.d/ssl directory.
4. Intend to use nginx to provide wss proxy services on port 443 (the port can be modified as needed).
5. Since nginx generally serves as a website server, to avoid affecting the original site usage, the address /wss is used as the proxy entrance for wss. That is, the client connection address is wss://domain.com/wss.

**Sample nginx configuration:**
```nginx
server {
  listen 443;
  # Domain configuration is omitted...

  ssl on;
  ssl_certificate /etc/ssl/server.pem;
  ssl_certificate_key /etc/ssl/server.key;
  ssl_session_timeout 5m;
  ssl_session_cache shared:SSL:50m;
  ssl_protocols SSLv3 SSLv2 TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;

  location /wss {
    proxy_pass http://127.0.0.1:8282;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
    proxy_set_header X-Real-IP $remote_addr;
  }
  
  # location / {} Other configurations for the site...
}
```

**Testing:**
```javascript
// The certificate will validate the domain, use the domain to connect. Note that the port is not included here
ws = new WebSocket("wss://domain.com/wss");

ws.onopen = function() {
    alert("Connection established");
    ws.send('tom');
    alert("Sent a string to the server: tom");
};
ws.onmessage = function(e) {
    alert("Received a message from the server: " + e.data);
};
```

## Using Apache to Proxy wss

Apache can also be used to proxy wss and redirect it to Workerman.

**Preparation:**

1. GatewayWorker is listening on port 8282 (websocket protocol).
2. SSL certificate has been obtained and placed in /server/httpd/cert/ directory.
3. Use Apache to forward port 443 to the specified port 8282.
4. httpd-ssl.conf has been loaded.
5. openssl is installed.

**Enable the proxy_wstunnel_module module:**
```apache
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_wstunnel_module modules/mod_proxy_wstunnel.so
```

**SSL and Proxy Configuration:**
```apache
#extra/httpd-ssl.conf
DocumentRoot "/website/directory"
ServerName domain

# Proxy Config
SSLProxyEngine on

ProxyRequests Off
ProxyPass /wss ws://127.0.0.1:8282/wss
ProxyPassReverse /wss ws://127.0.0.1:8282/wss

# Add SSL protocol support, remove insecure protocols
SSLProtocol all -SSLv2 -SSLv3
# Modify the encryption suite as follows
SSLCipherSuite HIGH:!RC4:!MD5:!aNULL:!eNULL:!NULL:!DH:!EDH:!EXP:+MEDIUM
SSLHonorCipherOrder on
# Certificate public key configuration
SSLCertificateFile /server/httpd/cert/your.pem
# Certificate private key configuration
SSLCertificateKeyFile /server/httpd/cert/your.key
# Certificate chain configuration
SSLCertificateChainFile /server/httpd/cert/chain.pem
```

**Testing:**
```javascript
// The certificate will validate the domain, use the domain to connect. Note that there is no port
ws = new WebSocket("wss://domain.com/wss");

ws.onopen = function() {
    alert("Connection established");
    ws.send('tom');
    alert("Sent a string to the server: tom");
};
ws.onmessage = function(e) {
    alert("Received a message from the server: " + e.data);
};
```

## Method 2: Enable SSL Directly with Workerman (Not Recommended)

> **Note:**
> Either use nginx/apache to proxy SSL or enable SSL using Workerman, but not both at the same time.

**Preparation:**

1. Workerman version >= 3.3.7.
2. PHP installed with the openssl extension.
3. SSL certificate (pem/crt file and key file) obtained and placed in any directory.

**Code:**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// It is recommended to use a valid certificate
$context = array(
    'ssl' => array(
        'local_cert' => 'disk_path/server.pem', // Alternatively, it can be a crt file
        'local_pk' => 'disk_path/server.key',
        'verify_peer' => false,
        'allow_self_signed' => true, 
    )
);
// Use websocket protocol here (port can be any, but ensure it's not used by other programs)
$worker = new Worker('websocket://0.0.0.0:8282', $context);
// Enable ssl for transport, websocket + ssl as wss
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

With the above code, Workerman listens to the wss protocol, and clients can connect to Workerman to achieve secure instant communication using the wss protocol.

**Testing:**

Open the Chrome browser, press F12 to open the debug console, and in the Console column, enter the following code (or place the code in an HTML page and run it using JavaScript):

```javascript
// The certificate will validate the domain, use the domain to connect, note the port included here
ws = new WebSocket("wss://domain.com:8282");
ws.onopen = function() {
    alert("Connection established");
    ws.send('tom');
    alert("Sent a string to the server: tom");
};
ws.onmessage = function(e) {
    alert("Received a message from the server: " + e.data);
};
```

**Notes:**

1. If it is necessary to use port 443, use the first method of nginx/apache proxy to implement wss.
2. wss ports can only be accessed via the wss protocol; ws cannot access wss ports.
3. Certificates are generally bound to domains, so when testing, the client should use the domain to connect, not an IP address.
4. If there are any issues accessing, please check the server's firewall.
5. This method requires PHP version >= 5.6, as WeChat Mini Program requires tls1.2, and PHP versions below 5.6 do not support tls1.2.

Related Articles:  
[Get Real IP from Proxy](get-real-ip-from-proxy.md)  
[Workerman SSL Context Options Reference](https://php.net/manual/en/context.ssl.php)
