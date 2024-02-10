# Erstellen Sie einen HTTPS-Dienst

**Frage:**

Wie erstellt man einen HTTPS-Dienst mit Workerman, so dass sich Client über das HTTPS-Protokoll verbinden können?

**Antwort:**

Das HTTPS-Protokoll ist tatsächlich eine Kombination aus HTTP und SSL, d.h. es fügt der HTTP-Protokollschicht eine SSL-Ebene hinzu. Workerman unterstützt das HTTP-Protokoll und ab der Version 3.3.7 auch SSL. Daher kann man durch Aktivierung von SSL auf der Grundlage des HTTP-Protokolls auch das HTTPS-Protokoll nutzen.

Es gibt zwei gängige Methoden, um Workerman für HTTPS zu konfigurieren: Entweder direkt SSL in Workerman aktivieren oder SSL über einen NGINX-Proxy verwenden. Es kann nur eine der beiden Methoden gleichzeitig verwendet werden, nicht beide.

## Aktivierung von SSL in Workerman

**Vorbereitung:**

1. Workerman-Version >= 3.3.7
2. PHP mit installierter OpenSSL-Erweiterung
3. Zertifikat im PEM/CRT-Format und Key-Datei im Verzeichnis /etc/nginx/conf.d/ssl

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Es wird empfohlen, ein gültiges Zertifikat zu verwenden
$context = array(
    'ssl' => array(
        'local_cert'        => '/etc/nginx/conf.d/ssl/server.pem', // kann auch eine CRT-Datei sein
        'local_pk'          => '/etc/nginx/conf.d/ssl/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // Bei selbst signierten Zertifikaten diese Option aktivieren
    )
);
// Hier wird das HTTP-Protokoll verwendet
$worker = new Worker('http://0.0.0.0:443', $context);
// Aktivierung von SSL als Transport, um HTTP+SSL (d.h. HTTPS) zu ermöglichen
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

Der obige Code erstellt einen HTTPS-Dienst mit Workerman, über den sich Client sicher und verschlüsselt verbinden können.

**Test:**

Geben Sie in die Adressleiste des Browsers "https://domain.com:443" ein, um darauf zuzugreifen.

**Hinweis:**

1. Der HTTPS-Port erfordert den Zugriff über das HTTPS-Protokoll, nicht über HTTP.
2. Zertifikate sind normalerweise an Domains gebunden, daher ist es beim Testen wichtig, die Domain zu verwenden und nicht die IP-Adresse.
3. Wenn der HTTPS-Zugriff nicht funktioniert, sollte die Server-Firewall überprüft werden.

## Verwendung von NGINX als SSL-Proxy

Abgesehen von der direkten SSL-Aktivierung in Workerman kann auch NGINX als SSL-Proxy verwendet werden, um HTTPS zu implementieren.

> **Wichtig**
> Die Verwendung  des NGINX SSL-Proxys und die SSL-Konfiguration in Workerman sind alternative Methoden und können nicht gleichzeitig aktiviert werden.

Die Kommunikationsprinzipien und Abläufe sind wie folgt:

1. Der Client stellt über HTTPS eine Verbindung zum NGINX her.
2. Der NGINX konvertiert die Daten aus dem HTTPS-Protokoll in das HTTP-Protokoll und leitet sie an den HTTP-Port von Workerman weiter.
3. Workerman empfängt die Daten, führt die Geschäftslogik aus und sendet die HTTP-Daten zurück an den NGINX.
4. Der NGINX konvertiert die HTTP-Daten wieder in HTTPS und leitet sie an den Client weiter.

### Beispiel für die NGINX-Konfiguration

**Voraussetzungen und Vorbereitungen:**

1. Angenommen, Workerman hört auf Port 8181 (HTTP-Protokoll)
2. Zertifikate im PEM/CRT-Format und Key-Datei im Verzeichnis /etc/nginx/conf.d/ssl
3. Sie planen, den Port 443 für den externen WSS-Proxydienst zu verwenden (der Port kann je nach Bedarf angepasst werden)

**Eine beispielhafte NGINX-Konfiguration sieht wie folgt aus:**

```nginx
upstream workerman {
    server 127.0.0.1:8181;
    keepalive 10240;
}

server {
  listen 443;
  server_name domain.com;
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
