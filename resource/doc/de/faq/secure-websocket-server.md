# Erstellen eines WSS-Dienstes

**Frage:**

Wie erstelle ich einen WSS-Dienst in Workerman, so dass Clients über das WSS-Protokoll eine Verbindung herstellen und kommunizieren können, z. B. in einer WeChat Mini-App, um eine Verbindung zum Server herzustellen?

**Antwort:**

Das WSS-Protokoll ist tatsächlich eine Kombination aus [WebSocket](https://baike.baidu.com/item/WebSocket) und [SSL](https://baike.baidu.com/item/ssl), das heißt, es wird eine SSL-Schicht über das WebSocket-Protokoll gelegt, ähnlich wie bei [HTTPS](https://baike.baidu.com/item/https) ([HTTP](https://baike.baidu.com/item/http)+[SSL](https://baike.baidu.com/item/ssl)). Daher ist es lediglich erforderlich, die SSL-Ebene auf dem WebSocket-Protokoll zu aktivieren, um das WSS-Protokoll zu unterstützen.

## Methode 1: Verwendung von Nginx/Apache zur SSL-Proxy-Umleitung (empfohlen)

**Kommunikationsprinzip und Ablauf**

1. Der Client stellt eine WSS-Verbindung mit Nginx/Apache her.

2. Nginx/Apache wandelt die WSS-Protokolldaten in WS-Protokolldaten um und leitet sie an den WebSocket-Port von Workerman weiter.

3. Workerman verarbeitet die empfangenen Daten und führt die Geschäftslogik aus.

4. Wenn Workerman eine Nachricht an den Client sendet, erfolgt der umgekehrte Prozess: Die Daten werden von Nginx/Apache in das WSS-Protokoll umgewandelt und an den Client gesendet.

## Nginx-Konfigurationsreferenz

**Voraussetzungen und Vorbereitungen:**

1. Nginx ist bereits installiert, Version ist nicht niedriger als 1.3.

2. Angenommen, Workerman lauscht auf Port 8282 (WebSocket-Protokoll).

3. Sie haben ein Zertifikat (PEM/CRT-Datei und Schlüsseldatei) beantragt und diese im Verzeichnis /etc/nginx/conf.d/ssl abgelegt.

4. Sie möchten den Port 443 für den WSS-Proxy-Service (der Port kann je nach Bedarf geändert werden) öffnen.

5. Nginx läuft normalerweise als Webserver für andere Dienste. Um die bestehende Website nicht zu beeinträchtigen, verwenden Sie den Pfad `domain.com/wss` als Einstiegspunkt für den WSS-Proxy. Dies bedeutet, dass die Client-Verbindungsadresse wss://domain.com/wss lautet.

**Nginx-Konfiguration ähnlich wie folgt**:
```nginx
server {
  listen 443;
  # Domain-Konfiguration ausgelassen...

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
    proxy_set_header X-Real-IP $remote_addr;
  }
  
  # location / {} Konfiguration für andere Standorte...
}
```
**Test**
```javascript
// Das Zertifikat prüft die Domain; verwenden Sie daher die Domain-Verbindung ohne den Port.
ws = new WebSocket("wss://domain.com/wss");

ws.onopen = function() {
    alert("Verbindung hergestellt");
    ws.send('tom');
    alert("Sende dem Server einen String: tom");
};
ws.onmessage = function(e) {
    alert("Nachricht vom Server erhalten: " + e.data);
};
```

## Verwendung von Apache zur WSS-Proxy-Umleitung

Sie können auch Apache verwenden, um den WSS an Workerman weiterzuleiten.

Vorbereitung:

1. GatewayWorker lauscht auf Port 8282 (WebSocket-Protokoll).

2. Sie haben ein SSL-Zertifikat beantragt und in /server/httpd/cert/ abgelegt.

3. Leiten Sie den Port 443 an den Port 8282 um.

4. httpd-ssl.conf wurde geladen.

5. OpenSSL ist installiert.

**Aktivieren Sie das Modul für die Umleitung von Proxy_WSTunnel**
```apache
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_wstunnel_module modules/mod_proxy_wstunnel.so
```

**SSL- und Proxy-Konfiguration**
```apache
#extra/httpd-ssl.conf
DocumentRoot "/Pfad/zur/Website"
ServerName Domain

# Proxy-Konfiguration
SSLProxyEngine on

ProxyRequests Off
ProxyPass /wss ws://127.0.0.1:8282/wss
ProxyPassReverse /wss ws://127.0.0.1:8282/wss

# SSL-Protokollunterstützung hinzufügen und unsichere Protokolle entfernen
SSLProtocol all -SSLv2 -SSLv3
# Aktualisierung der Verschlüsselungssuite
SSLCipherSuite HIGH:!RC4:!MD5:!aNULL:!eNULL:!NULL:!DH:!EDH:!EXP:+MEDIUM
SSLHonorCipherOrder on
# Konfigurieren des Zertifikatsschlüssels
SSLCertificateFile /server/httpd/cert/your.pem
# Konfigurieren des privaten Zertifikatsschlüssels
SSLCertificateKeyFile /server/httpd/cert/your.key
# Konfigurieren der Zertifikatskette
SSLCertificateChainFile /server/httpd/cert/chain.pem
```

**Test**
```javascript
// Das Zertifikat prüft die Domain; verwenden Sie daher die Domain-Verbindung ohne den Port.
ws = new WebSocket("wss://domain.com/wss");

ws.onopen = function() {
    alert("Verbindung hergestellt");
    ws.send('tom');
    alert("Sende dem Server einen String: tom");
};
ws.onmessage = function(e) {
    alert("Nachricht vom Server erhalten: " + e.data);
};
```

## Methode 2: Direkte Aktivierung von SSL über Workerman (nicht empfohlen)

> **Hinweis:**
> Die SSL-Umleitung über Nginx/Apache und das SSL-Setup über Workerman sind alternative Optionen und dürfen nicht gleichzeitig aktiviert sein.

**Vorbereitung:**

1. Workerman-Version >= 3.3.7

2. PHP mit aktiviertem OpenSSL-Modul

3. Ein Zertifikat (PEM/CRT-Datei und Schlüsseldatei) wurde auf einem beliebigen Laufwerk abgelegt.

**Code:**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$context = array(
    'ssl' => array(
        'local_cert'        => 'Pfad/server.pem', 
        'local_pk'          => 'Pfad/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true,
    )
);

$worker = new Worker('websocket://0.0.0.0:8282', $context);
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

Mit diesem Code lauscht Workerman auf dem WSS-Protokoll, und Clients können über das WSS-Protokoll eine sichere Echtzeitkommunikation mit Workerman herstellen.

**Test**

Öffnen Sie den Google Chrome und drücken Sie F12, um die Entwicklerkonsole zu öffnen. Geben Sie im Konsolenbereich Folgendes ein (oder fügen Sie den folgenden Code in eine HTML-Seite ein und führen Sie ihn mit JavaScript aus):

```javascript
// Das Zertifikat prüft die Domain; verwenden Sie daher die Domain-Verbindung mit dem Port.
ws = new WebSocket("wss://domain.com:8282");
ws.onopen = function() {
    alert("Verbindung hergestellt");
    ws.send('tom');
    alert("Sende dem Server einen String: tom");
};
ws.onmessage = function(e) {
    alert("Nachricht vom Server erhalten: " + e.data);
};
```

**Anmerkungen:**

1. Wenn Port 443 erforderlich ist, verwenden Sie die erste Methode (Nginx/Apache-Proxy) zur Implementierung von WSS.

2. Der WSS-Port kann nur über das WSS-Protokoll zugegriffen werden. Das WS-Protokoll kann nicht auf den WSS-Port zugreifen.

3. Zertifikate sind in der Regel an Domains gebunden. Verwenden Sie daher die Domain-Verbindung für Tests und vermeiden Sie die Verwendung von IP-Adressen.

4. Überprüfen Sie die Firewall auf dem Server, falls der Zugriff nicht möglich ist.

5. Diese Methode erfordert PHP-Versionen >= 5.6, da WeChat Mini-Programme TLS 1.2 erfordern, und PHP-Versionen vor 5.6 TLS 1.2 nicht unterstützen.

Verwandte Artikel:
[Erhalten der echten IP-Adresse von einem Proxy](get-real-ip-from-proxy.md)
[Referenz für SSL-Kontextoptionen in Workerman](https://php.net/manual/zh/context.ssl.php)
