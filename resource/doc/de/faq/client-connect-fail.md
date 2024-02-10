# Gründe für den fehlgeschlagenen Client-Verbindung

In der Regel gibt es zwei Arten von Fehlern, die bei einer fehlgeschlagenen Client-Verbindung auftreten können: ```connection refuse``` (Verbindung abgelehnt) und ```connection timeout``` (Verbindungszeitüberschreitung).

## Verbindung abgelehnt (connection refuse)

In der Regel tritt dies aus folgenden Gründen auf:
1. Der Client hat den falschen Port für die Verbindung verwendet.
2. Der Client hat den falschen Hostnamen oder die falsche IP-Adresse für die Verbindung verwendet.
3. Wenn der Client einen Domänennamen verwendet, könnte dieser auf die falsche Server-IP-Adresse zeigen.
4. Der Server verwendet CDN oder andere Beschleunigungsproxy, was dazu führt, dass die tatsächliche IP-Adresse nicht mit der erwarteten IP-Adresse übereinstimmt.
5. Der Server ist nicht gestartet oder der Port wird nicht überwacht.
6. Es werden Netzwerkproxy-Software verwendet.
7. Die Serverüberwachungs-IP und die Zugriffsadresse sind nicht im selben Adressbereich. Zum Beispiel, wenn der Server auf 127.0.0.1 überwacht, kann der Client nur über 127.0.0.1 und nicht über die IP-Adresse des lokalen Netzwerks oder des öffentlichen Netzwerks eine Verbindung herstellen. Es wird empfohlen, die Überwachungsadresse auf 0.0.0.0 zu setzen, damit sowohl lokale, lokale Netzwerk- als auch öffentliche Netzwerkverbindungen hergestellt werden können.

## Verbindungszeitüberschreitung (connection timeout)

In der Regel tritt dies aus folgenden Gründen auf:
1. Die Serverfirewall blockiert die Verbindung. Probieren Sie, die Firewall vorübergehend zu deaktivieren.
2. Bei einem Cloud-Server können Sicherheitsgruppen die Verbindung blockieren. Sie müssen die entsprechenden Ports im Administrationsbereich öffnen.
3. Wenn Sie Control Panels wie Plesk verwenden, müssen Sie die entsprechenden Ports in Plesk öffnen.
4. Der Server existiert nicht oder wurde nicht gestartet.
5. Wenn der Client einen Domänennamen verwendet, könnte dieser auf die falsche Server-IP-Adresse zeigen.
6. Der Client greift auf die interne IP-Adresse des Servers zu und das Client- und Servergerät befinden sich nicht im selben lokalen Netzwerk.

## Kann die angeforderte Adresse nicht zuweisen (cannot assign requested address)

**Wenn Sie als Client** fungieren, müssen Sie für jede Verbindung einen temporären lokalen Port auf Ihrem Gerät verwenden. Eine Standardserver kann in der Regel ungefähr 20.000 bis 30.000 temporäre Ports verwenden. Wenn die Anzahl der Verbindungen zu einem bestimmten Server diesen Wert übersteigt, kann keine verfügbare Portnummer zugewiesen werden, und es kommt zu diesem Fehler. Sie können die Anzahl der lokalen temporären Ports erhöhen, indem Sie den Kernelparameter `/etc/sysctl.conf` `net.ipv4.ip_local_port_range` ändern. Zum Beispiel können Sie es auf `10000 65535` setzen (der Bereich der lokalen Ports wird auf 10000 bis 65535 erhöht, was bedeutet, dass die Anzahl der lokalen Ports auf 55535 erhöht wird). Führen Sie dann `sysctl -p` aus, um dies zu aktivieren.

Darüber hinaus bleibt die Verbindung nach dem Trennen für eine bestimmte Zeit im TIME_WAIT-Status und belegt weiterhin den entsprechenden lokalen Port. Kurz gesagt, wenn in kurzer Zeit viele (über 20.000 bis 30.000) kurzlebige Verbindungen hergestellt werden, wird ebenfalls ein `Cannot assign requested address` Fehler auftreten. In diesem Fall können Sie das schnelle Zurücksetzen des TIME_WAIT-Status im Kernel einstellen, um das Problem zu lösen. Näheres dazu finden Sie unter [Kerneloptimierung](https://www.workerman.net/doc/workerman/appendices/kernel-optimization.html).

> **Hinweis**
> Die Begrenzung der Anzahl lokaler Ports gilt nur für Clients. Auf Serverseite gibt es keine Begrenzung der lokalen Ports. Solange genügend Ressourcen vorhanden sind, ist die Anzahl der aufrechterhaltenen Verbindungen auf Serverseite praktisch unbegrenzt.

## Andere Fehlermeldungen

Wenn die Fehlermeldung weder ```connection refuse``` noch ```connection timeout``` lautet, liegt dies in der Regel an folgenden Gründen:

**1. Der vom Client verwendete Kommunikationsprotokoll stimmt nicht mit dem Server überein.**
Wenn beispielsweise der Server das HTTP-Kommunikationsprotokoll verwendet, kann der Client bei Verwendung des Websocket-Kommunikationsprotokolls keine Verbindung herstellen. Wenn der Client das WebSocket-Protokoll verwendet, muss der Server ebenfalls das WebSocket-Protokoll verwenden. Wenn der Server ein HTTP-Protokolldienst ist, muss der Client das HTTP-Protokoll verwenden.

Das Prinzip ist ähnlich wie beim Kommunizieren mit einem Englischsprachigen, um Englisch zu verwenden, oder mit einem Japanischsprachigen, um Japanisch zu verwenden. Hier ist die Sprache ähnlich wie das Kommunikationsprotokoll, beide Seiten (Client und Server) müssen die gleiche Sprache verwenden, um miteinander zu kommunizieren, sonst ist keine Kommunikation möglich.

**Häufige Fehlermeldungen aufgrund inkonsistenter Kommunikationsprotokolle umfassen:**

> WebSocket-Verbindung zu 'ws://xxx.com:xx/' fehlgeschlagen: Fehler bei WebSocket-Handshake: Unerwarteter Antwortcode: xxx

> WebSocket-Verbindung zu 'ws://xxx.com:xx/' fehlgeschlagen: Fehler bei WebSocket-Handshake: net::ERR_INVALID_HTTP_RESPONSE

**Lösung:**
Basierend auf den oben genannten Fehlermeldungen verwendet der Client eine ws-Verbindung, was auf das WebSocket-Protokoll hinweist. Der Server muss auch das WebSocket-Protokoll verwenden, um zu kommunizieren. Der folgende Beispielcode zeigt, wie der Serverbereich für das Websocket-Protokoll festgelegt werden kann:

Wenn es sich um GatewayWorker handelt, lautet der Code für den Überwachungsbereich beispielsweise wie folgt:

```php
// WebSocket-Protokoll, damit der Client eine Verbindung mit ws://... herstellen kann. xxxx ist der Port und muss nicht geändert werden
$gateway = new Gateway('websocket://0.0.0.0:xxxx');
```

Bei Workerman lautet der entsprechende Code wie folgt:

```php
// WebSocket-Protokoll, damit der Client eine Verbindung mit ws://... herstellen kann. xxxx ist der Port und muss nicht geändert werden
$worker = new Worker('websocket://0.0.0.0:xxxx');
```
