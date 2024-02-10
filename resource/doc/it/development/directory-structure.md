# Struttura delle directory
```
Workerman                      // Codice sorgente del kernel di workerman
    ├── Connection                 // Connessioni socket correlate
    │   ├── ConnectionInterface.php// Interfaccia di connessione socket
    │   ├── TcpConnection.php      // Classe di connessione Tcp
    │   ├── AsyncTcpConnection.php // Classe di connessione Tcp asincrona
    │   └── UdpConnection.php      // Classe di connessione Udp
    ├── Events                     // Libreria eventi di rete
    │   ├── EventInterface.php     // Interfaccia della libreria eventi di rete
    │   ├── Event.php              // Libreria eventi di rete Libevent
    │   ├── Ev.php                 // Libreria eventi di rete Libev
    │   ├── Swoole.php             // Libreria eventi di rete Swoole
    │   └── Select.php             // Libreria eventi di rete Select
    ├── Lib                        // Librerie comuni
    │   ├── Constants.php          // Definizione costanti
    │   └── Timer.php              // Timer
    ├── Protocols                  // Protocolli correlati
    │   ├── ProtocolInterface.php  // Classe di interfaccia del protocollo
    │   ├── Http                   // Protocollo http correlato
    │   │   ├── Chunk.php    // Classe chunk http
    │   │   ├── Request.php  // Classe di richiesta http
    │   │   ├── Response.php  // Classe di risposta http
    │   │   ├── ServerSentEvents.php  // Classe SSE
    │   │   ├── Session
    │   │   │   ├── FileSessionHandler.php  // Archiviazione sessione file
    │   │   │   └── RedisSessionHandler.php // Archiviazione sessione redis
    │   │   ├── Session.php  // Classe sessione
    │   │   └── mime.types   // File di mappatura mime
    │   ├── Http.php               // Implementazione protocollo http
    │   ├── Text.php               // Implementazione protocollo Text
    │   ├── Frame.php              // Implementazione protocollo Frame
    │   └── Websocket.php          // Implementazione protocollo websocket
    ├── Worker.php                 // Worker
    ├── WebServer.php              // Server web
    └── Autoloader.php             // Caricatore automatico classi
```
