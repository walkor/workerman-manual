# SSL/TLS Encryption for Data Transmission

**Question:**

How can the security of communication between Workerman and clients be ensured?

**Answer:**

A convenient approach is to add a layer of [SSL](https://baike.baidu.com/item/ssl) encryption on top of the communication protocol. For example, protocols like wss and [https](https://baike.baidu.com/item/https) are both based on [SSL](https://baike.baidu.com/item/ssl) encryption and are considered highly secure. Workerman itself supports [SSL](https://baike.baidu.com/item/ssl) (requires Workerman>=3.3.7), and SSL can be easily enabled by setting a few attributes.

Developers can also implement their own encryption and decryption mechanisms based on certain cryptographic algorithms.

## Method for Enabling SSL in Workerman:

**Prerequisites:**

1. Workerman version is at least 3.3.7.
2. PHP has the openssl extension installed.
3. Certificate (pem/crt file and key file) is obtained and placed in /etc/nginx/conf.d/ssl directory.

**Code:**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// It is recommended to use an obtained SSL certificate.
$context = [
    'ssl' => [
        'local_cert'        => '/etc/nginx/conf.d/ssl/server.pem', // Can also be a .crt file
        'local_pk'          => '/etc/nginx/conf.d/ssl/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // Enable this option for self-signed certificates
    ]
];
// Set the protocol to be websocket, but it can also be http or other protocols
$worker = new Worker('websocket://0.0.0.0:443', $context);
// Enable SSL for transport
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

## Enabling Server Name Indication (SNI) in Workerman

SNI allows binding multiple certificates to the same IP and port.
  
**Merging .pem and .key Files:**

Combine the content of the .pem and corresponding .key files for each certificate by adding the content of the .key file to the end of the .pem file. (If the .pem file already includes the private key, this step can be ignored.)

**Please note that it should be a single certificate, not all certificates copied into one file.**

For example, the combined content of host1.com.pem after merging may look like this:

```text
-----BEGIN CERTIFICATE-----
MIIGXTCBA...
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
MIIFBzCCA...
-----END CERTIFICATE-----
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAA....
-----END RSA PRIVATE KEY-----
```

**Code:**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$context = [
    'ssl' => [
        'SNI_enabled' => true, // Enable SNI
        'SNI_server_certs' => [ // Set multiple certificates
            'host1.com' => '/path/host1.com.pem', // Certificate 1
            'host2.com' => '/path/host2.com.pem', // Certificate 2
        ],
        'local_cert' => '/path/default.com.pem', // Default certificate
        'local_pk'   => '/path/default.com.key',
    ]
];
$worker = new Worker('websocket://0.0.0.0:443', $context);
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```
