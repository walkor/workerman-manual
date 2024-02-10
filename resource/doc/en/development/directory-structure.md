# Directory Structure
```
Workerman                      // Workerman core code
    ├── Connection                 // Socket connection related
    │   ├── ConnectionInterface.php// Socket connection interface
    │   ├── TcpConnection.php      // TCP connection class
    │   ├── AsyncTcpConnection.php // Asynchronous TCP connection class
    │   └── UdpConnection.php      // UDP connection class
    ├── Events                     // Network event libraries
    │   ├── EventInterface.php     // Network event library interface
    │   ├── Event.php              // Libevent network event library
    │   ├── Ev.php                 // Libev network event library
    │   ├── Swoole.php             // Swoole network event library
    │   └── Select.php             // Select network event library
    ├── Lib                        // Common libraries
    │   ├── Constants.php          // Constant definitions
    │   └── Timer.php              // Timer
    ├── Protocols                  // Protocol related
    │   ├── ProtocolInterface.php  // Protocol interface class
    │   ├── Http                   // HTTP protocol related
    │   │   ├── Chunk.php    // HTTP chunk class
    │   │   ├── Request.php  // HTTP request class
    │   │   ├── Response.php  // HTTP response class
    │   │   ├── ServerSentEvents.php  // SSE class
    │   │   ├── Session
    │   │   │   ├── FileSessionHandler.php  // Session file storage
    │   │   │   └── RedisSessionHandler.php // Session Redis storage
    │   │   ├── Session.php  // Session class
    │   │   └── mime.types   // Mime type mapping file
    │   ├── Http.php               // HTTP protocol implementation
    │   ├── Text.php               // Text protocol implementation
    │   ├── Frame.php              // Frame protocol implementation
    │   └── Websocket.php          // Websocket protocol implementation
    ├── Worker.php                 // Worker
    ├── WebServer.php              // WebServer
    └── Autoloader.php             // Autoload class
```
