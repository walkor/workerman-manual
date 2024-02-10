# Transport Encryption - SSL/TLS

**Frage:**

Wie kann die Sicherheit der Kommunikation zwischen webman und Workerman sichergestellt werden?

**Antwort:**

Eine praktische Methode ist die Hinzufügung einer Schicht von [SSL](https://baike.baidu.com/item/ssl)-Verschlüsselung auf der Kommunikationsprotokollebene. Zum Beispiel basieren wss- und [https](https://baike.baidu.com/item/https)-Protokolle auf SSL-Verschlüsselung und sind daher äußerst sicher. Workerman unterstützt SSL ([Workerman-Version>=3.3.7 erforderlich](https://baike.baidu.com/item/ssl)), und es ist nur notwendig, die entsprechenden Einstellungen vorzunehmen, um SSL zu aktivieren.

Selbstverständlich können Entwickler auch basierend auf bestimmten Verschlüsselungsalgorithmen einen eigenen Verschlüsselungsmechanismus implementieren.

## Aktivierung von SSL in Workerman:

**Vorbereitung:**

1. Workerman-Version muss mindestens 3.3.7 betragen.
2. PHP muss die openssl-Erweiterung installiert haben.
3. Ein Zertifikat (pem/crt-Datei und Schlüsseldatei) sollte im Verzeichnis /etc/nginx/conf.d/ssl abgelegt sein.

**Code:**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$context = array(
    'ssl' => array(
        'local_cert'        => '/etc/nginx/conf.d/ssl/server.pem', // oder .crt-Datei
        'local_pk'          => '/etc/nginx/conf.d/ssl/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // Bei selbst signierten Zertifikaten diese Option aktivieren
    )
);

$worker = new Worker('websocket://0.0.0.0:443', $context);
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

## Aktivierung des Server Name Indication (SNI) in Workerman

Dadurch können unter derselben IP-Adresse und derselben Portnummer mehrere Zertifikate gebunden werden.

**Zusammenführen von .pem- und .key-Dateien:**

Die Inhalte jeder .pem-Datei und der entsprechenden .key-Datei werden zusammengeführt, indem der Inhalt der .key-Datei an das Ende der .pem-Datei angefügt wird. (Wenn die .pem-Datei bereits den privaten Schlüssel enthält, kann dieser Schritt ignoriert werden.)

**Bitte beachten Sie, dass es sich um einzelne Zertifikate handelt und nicht um das Kopieren aller Zertifikate in eine Datei.**

Beispielinhalt der zusammengeführten .pem-Datei nach dem Zusammenführen von *host1.com.pem* könnte wie folgt aussehen:

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

$context = array(
    'ssl' => array(
        'SNI_enabled' => true, // SNI aktivieren
        'SNI_server_certs' => [ // Mehrere Zertifikate einstellen
            'host1.com' => '/path/host1.com.pem', // Zertifikat 1
            'host2.com' => '/path/host2.com.pem', // Zertifikat 2
        ],
        'local_cert' => '/path/default.com.pem', // Standardzertifikat
        'local_pk'   => '/path/default.com.key',
    )
);

$worker = new Worker('websocket://0.0.0.0:443', $context);
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```
