# Creating HTTPS Service

**Question:**

How can Workerman create an HTTPS service so that clients can communicate using the HTTPS protocol?

**Answer:**

The HTTPS protocol is actually HTTP with SSL, adding an SSL layer on top of the HTTP protocol. Workerman supports both HTTP and SSL (`required Workerman version >= 3.3.7`), so enabling SSL on top of the HTTP protocol makes it possible to support the HTTPS protocol.

There are two common approaches to enable Workerman to support HTTPS: one is to directly enable SSL in Workerman, and the other is to use nginx as an SSL proxy. Only one of these approaches should be used at a time, they cannot be enabled simultaneously.

## Enabling SSL in Workerman

**Prerequisites:**

1. Workerman version >= 3.3.7
2. PHP with the openssl extension installed
3. Certificate (pem/crt file and key file) placed in `/etc/nginx/conf.d/ssl`

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Ideally use an acquired certificate
$context = array(
    'ssl' => array(
        'local_cert'        => '/etc/nginx/conf.d/ssl/server.pem', // Can also be a crt file
        'local_pk'          => '/etc/nginx/conf.d/ssl/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // Enable this for a self-signed certificate
    )
);
// Here we set the protocol as HTTP
$worker = new Worker('http://0.0.0.0:443', $context);
// Set the transport to enable SSL, making it HTTP+SSL, or HTTPS
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

With the above code, Workerman has created an HTTPS service, allowing clients to securely connect to Workerman using the HTTPS protocol.

**Testing:**

Enter `https://domain:443` in the browser's address bar to access the service.

**Note:**

1. The HTTPS port must be accessed using the HTTPS protocol; HTTP protocol will not work.
2. Certificates are typically tied to a domain, so use a domain for testing and avoid using an IP address.
3. If HTTPS access fails, check the server firewall.

## Using Nginx as an SSL Proxy

In addition to using SSL in Workerman itself, it is also possible to use nginx as an SSL proxy to enable HTTPS.

> **Note**
> Nginx SSL proxy and Workerman SSL configuration are mutually exclusive and cannot be enabled simultaneously.

The communication principle and flow are as follows:

1. The client initiates an HTTPS connection to nginx.
2. Nginx converts the HTTPS protocol data to HTTP and forwards it to Workerman's HTTP port.
3. Workerman receives the data, processes it, and returns HTTP protocol data to nginx.
4. Nginx then converts the HTTP protocol data back to HTTPS and forwards it to the client.

### Nginx Configuration Reference

**Prerequisites and Preparation:**

1. Assume that Workerman is listening on port 8181 for the HTTP protocol.
2. Certificate (pem/crt file and key file) placed in `/etc/nginx/conf.d/ssl`
3. Intention to use nginx to open port 443 for wss proxy service (modify the port as needed)

**Nginx configuration is similar to the following:**

```nginx
upstream workerman {
    server 127.0.0.1:8181;
    keepalive 10240;
}

server {
  listen 443;
  server_name example.com;
  access_log off;
  
  ssl on;
  ssl_certificate /etc/nginx/conf.d/ssl/server.pem;
  ssl_certificate_key /etc/nginx/conf.d/ssl/server.key;
  ssl_session_timeout 5m;
  ssl_session_cache shared:SSL:50m;
  ssl_protocols SSLv3 SSLv2 TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;

  location / {
    proxy_pass http://workerman;
    proxy_http_version 1.1;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Connection "";
  }
}
```
