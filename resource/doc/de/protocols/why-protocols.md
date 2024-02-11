# Zweck des Kommunikationsprotokolls
Da TCP auf einem Datenstrom basiert, fließen die vom Client gesendeten Anfragedaten wie Wasser in den Server. Nachdem der Server festgestellt hat, dass Daten eingetroffen sind, muss er prüfen, ob die Daten vollständig sind, da nur ein Teil einer Anfrage den Server erreichen kann. Es ist sogar möglich, dass mehrere Anfragen zusammen den Server erreichen. Um zu bestimmen, ob eine Anfrage vollständig eingegangen ist oder um Anfragen aus mehreren verbundenen Anfragen zu trennen, ist es erforderlich, ein Kommunikationsprotokoll festzulegen.

## Warum sollte in Workerman ein Protokoll festgelegt werden?
In der herkömmlichen PHP-Entwicklung basiert alles auf dem Web und im Wesentlichen auf dem HTTP-Protokoll. Die Analyse und Verarbeitung des HTTP-Protokolls erfolgt ausschließlich durch den Webserver, sodass sich Entwickler keine Gedanken über protokollbezogene Angelegenheiten machen müssen. Wenn jedoch eine Entwicklung auf der Grundlage eines nicht-HTTP-Protokolls erforderlich ist, müssen Entwickler nun Protokollangelegenheiten berücksichtigen.

## Unterstützte Protokolle in Workerman
Workerman unterstützt derzeit die Protokolle HTTP, Websocket, Text (siehe Anhang), Frame (siehe Anhang) und WS (siehe Anhang). Bei der Kommunikation über diese Protokolle kann direkt darauf zugegriffen werden. Die Verwendung erfolgt durch die Spezifikation des Protokolls bei der Initialisierung des Workers, beispielsweise so:
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// websocket://0.0.0.0:2345 zeigt an, dass der Port 2345 für das Websocket-Protokoll verwendet wird
$websocket_worker = new Worker('websocket://0.0.0.0:2345');

// Textprotokoll
$text_worker = new Worker('text://0.0.0.0:2346');

// Frame-Protokoll
$frame_worker = new Worker('frame://0.0.0.0:2347');

// TCP-Worker, direkt auf der Socket-Übertragung basierend, ohne Verwendung eines Anwendungsprotokolls
$tcp_worker = new Worker('tcp://0.0.0.0:2348');

// UDP-Worker, ohne Verwendung eines Anwendungsprotokolls
$udp_worker = new Worker('udp://0.0.0.0:2349');

// Unix-Domänen-Worker, ohne Verwendung eines Anwendungsprotokolls
$unix_worker = new Worker('unix:///tmp/wm.sock');
```

## Verwendung eines benutzerdefinierten Kommunikationsprotokolls
Wenn die vorinstallierten Kommunikationsprotokolle von Workerman den Entwicklungsanforderungen nicht gerecht werden, können Entwickler ihr eigenes Kommunikationsprotokoll anpassen. Die Anpassungsmethode finden Sie im nächsten Abschnitt.

**Hinweis:**
Workerman verfügt über ein integriertes Textprotokoll mit dem Format Text + Zeilenumbruch. Das Textprotokoll ist einfach für Entwicklung und Debugging, und unterstützt Telnet-Debugging. Wenn Entwickler ein eigenes Anwendungsprotokoll entwickeln möchten, können sie direkt das Textprotokoll verwenden, ohne ein separates entwickeln zu müssen.

Weitere Informationen zum Textprotokoll finden Sie im Anhang "Teil des Textprotokolls".
