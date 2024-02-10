# Wichtige Informationen für Workerman-Entwickler

**1. Einschränkungen in der Windows-Umgebung**

Unter dem Windows-Betriebssystem unterstützt ein einzelner Workerman-Prozess nur 200+ Verbindungen.
In der Windows-Umgebung kann die Anzahl der Prozesse nicht mit dem "count"-Parameter festgelegt werden.
In der Windows-Umgebung können Befehle wie "status", "stop", "reload", "restart" nicht verwendet werden.
In der Windows-Umgebung können keine Daemon-Prozesse erstellt werden, da der Dienst nach dem Schließen des CMD-Fensters gestoppt wird.
In der Windows-Umgebung kann nicht in einer Datei mehrere Überwachungspunkte initialisiert werden.
Für das Linux-Betriebssystem gelten diese Einschränkungen nicht. Es wird empfohlen, das Linux-Betriebssystem für die Produktionsumgebung zu verwenden und in der Entwicklungsphase kann das Windows-Betriebssystem verwendet werden.

**2. Workerman ist nicht von Apache oder Nginx abhängig**

Workerman selbst ist bereits ein Container ähnlich Apache/Nginx, solange die PHP-Umgebung OK ist, kann Workerman ausgeführt werden.

**3. Workerman wird über die Befehlszeile gestartet**

Die Startmethode ähnelt der Verwendung von Befehlen zum Starten von Apache (normalerweise nicht für Webhosting-Räume geeignet). Das Startfenster sieht ähnlich aus wie unten:
![](image/screenshot_1495622774534.png)

**4. Heartbeat ist ein Muss für Langzeitverbindungen**

Langzeitverbindungen müssen über Heartbeat verfügen. Es ist wichtig, Heartbeat zu verwenden, denn bei längerer Inaktivität werden die Verbindungen von Routing-Knoten gelöscht und geschlossen.
[Informationen zum Heartbeat in Workerman](faq/heartbeat.md), [Informationen zum Heartbeat in gatewayWorker](https://www.workerman.net/doc/gateway-worker/heartbeat.html)

**5. Client- und Serverprotokolle müssen übereinstimmen, um zu kommunizieren**

Dies ist ein sehr häufiges Problem bei Entwicklern. Wenn der Client z. B. das WebSocket-Protokoll verwendet, muss der Server auch das WebSocket-Protokoll verwenden (Der Server ```new Worker('websocket://0.0.0.0...')```) um verbunden werden und kommunizieren zu können.
Versuchen Sie nicht, über die Adressleiste des Browsers auf den WebSocket-Protokollport zuzugreifen, und versuchen Sie nicht, das WebSocket-Protokoll auf einem nackten TCP-Protokollport zu verwenden. Die Protokolle müssen übereinstimmen.

Dieses Prinzip ist ähnlich wie bei der Kommunikation mit einer Person aus Großbritannien, bei der die Verwendung von Englisch erforderlich ist. Wenn Sie mit jemandem aus Japan kommunizieren möchten, müssen Sie Japanisch sprechen. Die Sprache ähnelt hier dem Kommunikationsprotokoll. Beide Seiten (Client und Server) müssen dieselbe Sprache verwenden, um zu kommunizieren, andernfalls ist keine Kommunikation möglich.

**6. Mögliche Gründe für Verbindungsfehler**

Ein häufiges Problem bei der erstmaligen Verwendung von Workerman besteht darin, dass die Clientverbindung zum Server fehlschlägt. Die Gründe hierfür sind im Allgemeinen:
1. Die Server-Firewall (einschließlich der Sicherheitsgruppe in der Cloud-Server) blockiert die Verbindung (mit einer Wahrscheinlichkeit von 50%).
2. Der Client und der Server verwenden inkongruente Protokolle (mit einer Wahrscheinlichkeit von 30%).
3. Die IP oder Port wurde falsch eingegeben (mit einer Wahrscheinlichkeit von 15%).
4. Der Server wurde nicht gestartet.

**7. Verwenden Sie keine "exit", "die" oder "sleep"-Anweisungen**

Das Ausführen von "exit" oder "die" führt zum Beenden des Prozesses und es wird der Fehler "WORKER EXIT UNEXPECTED" angezeigt. Natürlich wird beim Beenden des Prozesses sofort ein neuer Prozess gestartet, um den Dienst fortzusetzen. Wenn eine Rückgabe erforderlich ist, verwenden Sie "return". Die "sleep"-Anweisung lässt den Prozess schlafen und während des Schlafens wird keine geschäftliche Tätigkeit ausgeführt. Das Framework wird auch gestoppt, was dazu führt, dass alle Clientanfragen dieses Prozesses nicht verarbeitet werden können.

**8. Verwenden Sie die Funktion pcntl_fork nicht**

`pcntl_fork` wird verwendet, um dynamisch neue Prozesse zu erstellen. Wenn "pcntl_fork" im Geschäftscode verwendet wird, kann dies zu nicht recyclbaren Waisenprozessen führen, was zu geschäftlichen Problemen führt. Die Verwendung von "pcntl_fork" im Geschäftsbereich wirkt sich auch auf die Verarbeitung von Ereignissen wie Verbindungen, Nachrichten, Verbindungsabbrüchen, Timern usw. aus, was zu unvorhersehbaren Problemen führen kann.

**9. Vermeiden Sie Endlosschleifen im Geschäftscode**

Vermeiden Sie Endlosschleifen im Geschäftscode, da dies dazu führen kann, dass die Kontrolle nicht an das Workerman-Framework zurückgegeben wird, was dazu führt, dass andere Clientnachrichten nicht empfangen und verarbeitet werden können.

**10. Der Code muss neu gestartet werden**

Workerman ist ein in-memory Framework. Um die Auswirkungen des neuen Codes zu sehen, muss Workerman neu gestartet werden.

**11. Lange Verbindungen sollten mit dem GatewayWorker-Framework verwendet werden**

Viele Entwickler verwenden Workerman, um Anwendungen mit **langen Verbindungen** zu entwickeln, wie z.B. für Echtzeitkommunikation, das Internet der Dinge usw. Für Anwendungen mit **langen Verbindungen** wird empfohlen, das GatewayWorker-Framework zu verwenden, das speziell auf der Grundlage von Workerman entwickelt wurde und die Entwicklung von Anwendungen mit langen Verbindungen im Hintergrund vereinfacht und benutzerfreundlicher gestaltet.

**12. Unterstützung für höhere Parallelität**

Wenn die Anzahl der gleichzeitigen Verbindungen in einer Anwendung über 1000 liegt, wird dringend empfohlen, das Linux-Kernel zu optimieren und die Event-Erweiterung zu installieren. 

