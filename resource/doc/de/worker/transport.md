# Transport
## Erklärung:
```php
string Worker::$transport
```

Legt das Transportprotokoll fest, das vom aktuellen Worker-Objekt verwendet wird. Derzeit werden nur drei Arten unterstützt: tcp, udp und ssl. Wenn nicht festgelegt, wird standardmäßig tcp verwendet.

``` Beachten: Für SSL ist Workerman-Version >= 3.3.7 erforderlich ```

## Beispiel 1

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:8484');
// Verwendet das UDP-Protokoll
$worker->transport = 'udp';
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('Hello');
};
// Startet den Worker
Worker::runAll();
```

## Beispiel 2

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Das Zertifikat ist am besten ein gültiges Zertifikat
$context = array(
    'ssl' => array(
        'local_cert' => '/etc/nginx/conf.d/ssl/server.pem', // Kann auch eine crt-Datei sein
        'local_pk'   => '/etc/nginx/conf.d/ssl/server.key',
    )
);
// Hier wird das WebSocket-Protokoll festgelegt
$worker = new Worker('websocket://0.0.0.0:4431', $context);
// Legt den Transport auf SSL, WebSocket + SSL wird als wss bezeichnet
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```
