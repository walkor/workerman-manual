# Structure du répertoire
```
Workerman                      // Code source principal de Workerman
    ├── Connection                 // Connexions socket 
    │   ├── ConnectionInterface.php// Interface de connexion socket
    │   ├── TcpConnection.php      // Classe de connexion Tcp
    │   ├── AsyncTcpConnection.php // Classe de connexion Tcp asynchrone
    │   └── UdpConnection.php      // Classe de connexion Udp
    ├── Events                     // Bibliothèque d'événements réseau
    │   ├── EventInterface.php     // Interface de la bibliothèque d'événements réseau
    │   ├── Event.php              // Bibliothèque d'événements réseau Libevent
    │   ├── Ev.php                 // Bibliothèque d'événements réseau Libev
    │   ├── Swoole.php             // Bibliothèque d'événements réseau Swoole
    │   └── Select.php             // Bibliothèque d'événements réseau Select
    ├── Lib                        // Bibliothèques couramment utilisées
    │   ├── Constants.php          // Définition des constantes
    │   └── Timer.php              // Minuterie
    ├── Protocols                  // Protocoles associés
    │   ├── ProtocolInterface.php  // Classe d'interface de protocole
    │   ├── Http                   // Protocole http
    │   │   ├── Chunk.php    // Classe Chunk http
    │   │   ├── Request.php  // Classe de requête http
    │   │   ├── Response.php  // Classe de réponse http
    │   │   ├── ServerSentEvents.php  // Classe SSE
    │   │   ├── Session
    │   │   │   ├── FileSessionHandler.php  // Stockage de session dans un fichier
    │   │   │   └── RedisSessionHandler.php // Stockage de session dans Redis
    │   │   ├── Session.php  // Classe de session
    │   │   └── mime.types   // Fichier de mappage des types mime
    │   ├── Http.php               // Implémentation du protocole http
    │   ├── Text.php               // Implémentation du protocole Text
    │   ├── Frame.php              // Implémentation du protocole Frame
    │   └── Websocket.php          // Implémentation du protocole websocket
    ├── Worker.php                 // Worker
    ├── WebServer.php              // Serveur Web
    └── Autoloader.php             // Chargeur automatique de classes
```
