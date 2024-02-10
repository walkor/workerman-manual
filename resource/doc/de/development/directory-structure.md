# Verzeichnisstruktur
``` 
Workerman                      // Workerman-Kerncode
    ├── Connection                 // Verbindung im Zusammenhang mit Sockets
    │   ├── ConnectionInterface.php// Schnittstelle für die Socket-Verbindung
    │   ├── TcpConnection.php      // TCP-Verbindungsklasse
    │   ├── AsyncTcpConnection.php // Asynchrone TCP-Verbindungsklasse
    │   └── UdpConnection.php      // UDP-Verbindungsklasse
    ├── Events                     // Netzwerkevents-Bibliothek
    │   ├── EventInterface.php     // Schnittstelle für Netzwerkevents-Bibliothek
    │   ├── Event.php              // Netzwerkevents-Bibliothek für Libevent
    │   ├── Ev.php                 // Netzwerkevents-Bibliothek für Libev
    │   ├── Swoole.php             // Netzwerkevents-Bibliothek für Swoole
    │   └── Select.php             // Netzwerkevents-Bibliothek für Select
    ├── Lib                        // Häufig verwendete Klassenbibliothek
    │   ├── Constants.php          // Konstantendefinitionen
    │   └── Timer.php              // Timer
    ├── Protocols                  // Protokollbezogen
    │   ├── ProtocolInterface.php  // Schnittstellenklasse für Protokolle
    │   ├── Http                   // HTTP-Protokollbezogen
    │   │   ├── Chunk.php          // HTTP-Chunk-Klasse
    │   │   ├── Request.php        // HTTP-Anforderungsklasse
    │   │   ├── Response.php       // HTTP-Antwortklasse
    │   │   ├── ServerSentEvents.php  // SSE-Klasse
    │   │   ├── Session
    │   │   │   ├── FileSessionHandler.php  // Sitzungsdateispeicher
    │   │   │   └── RedisSessionHandler.php // Sitzung-Speicher für Redis
    │   │   ├── Session.php        // Sitzungsklasse
    │   │   └── mime.types         // MIME-Zuordnungsdatei
    │   ├── Http.php               // Implementierung des HTTP-Protokolls
    │   ├── Text.php               // Implementierung des Text-Protokolls
    │   ├── Frame.php              // Implementierung des Frame-Protokolls
    │   └── Websocket.php          // Implementierung des Websocket-Protokolls
    ├── Worker.php                 // Worker
    ├── WebServer.php              // WebServer
    └── Autoloader.php             // Automatischer Klassenlader
```
