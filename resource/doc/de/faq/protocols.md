# Workerman unterstützte Protokolle

Workerman unterstützt eine Vielzahl von Protokollen auf der Schnittstellenebene, solange sie dem `ConnectionInterface`-Interface entsprechen (siehe den Abschnitt zur benutzerdefinierten Kommunikationsprotokollen).

Zur Vereinfachung für Entwickler bietet Workerman das HTTP-Protokoll, das WebSocket-Protokoll sowie ein sehr einfaches Textprotokoll, das für die binäre Übertragung von Rahmenprotokollen verwendet werden kann. Entwickler können diese Protokolle direkt verwenden, ohne zusätzliche Entwicklungsarbeit leisten zu müssen. Wenn diese Protokolle nicht den Anforderungen entsprechen, können Entwickler ihr eigenes Protokoll gemäß dem Abschnitt zur benutzerdefinierten Protokolle implementieren.

Entwickler können auch direkt auf die TCP- oder UDP-Protokolle aufbauen.

Beispiel für die Verwendung von Protokollen:
```php
// HTTP-Protokoll
$worker1 = new Worker('http://0.0.0.0:1221');
// WebSocket-Protokoll
$worker2 = new Worker('websocket://0.0.0.0:1222');
// Text-Protokoll (Telnet-Protokoll)
$worker3 = new Worker('text://0.0.0.0:1223');
// Rahmenprotokoll (für die binäre Übertragung)
$worker3 = new Worker('frame://0.0.0.0:1223');
// Direkte Verwendung des TCP-Protokolls
$worker4 = new Worker('tcp://0.0.0.0:1224');
// Direkte Verwendung des UDP-Protokolls
$worker5 = new Worker('udp://0.0.0.0:1225');
```
