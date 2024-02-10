# Estructura de directorios
```plaintext
Workerman                      // Código interno de workerman
    ├── Connection                 // Conexión de socket relacionada
    │   ├── ConnectionInterface.php// Interfaz de conexión de socket
    │   ├── TcpConnection.php      // Clase de conexión Tcp
    │   ├── AsyncTcpConnection.php // Clase de conexión Tcp asíncrona
    │   └── UdpConnection.php      // Clase de conexión Udp
    ├── Events                     // Biblioteca de eventos de red
    │   ├── EventInterface.php     // Interfaz de la biblioteca de eventos de red
    │   ├── Event.php              // Biblioteca de eventos de red Libevent
    │   ├── Ev.php                 // Biblioteca de eventos de red Libev
    │   ├── Swoole.php             // Biblioteca de eventos de red Swoole
    │   └── Select.php             // Biblioteca de eventos de red Select
    ├── Lib                        // Bibliotecas comunes
    │   ├── Constants.php          // Definición de constantes
    │   └── Timer.php              // Temporizador
    ├── Protocols                  // Protocolos relacionados
    │   ├── ProtocolInterface.php  // Clase de interfaz de protocolo
    │   ├── Http                   // Protocolo http relacionado
    │   │   ├── Chunk.php    // Clase chunk de http
    │   │   ├── Request.php  // Clase de solicitud http
    │   │   ├── Response.php  // Clase de respuesta http
    │   │   ├── ServerSentEvents.php  // Clase SSE
    │   │   ├── Session
    │   │   │   ├── FileSessionHandler.php  // Almacenamiento de sesión de archivo
    │   │   │   └── RedisSessionHandler.php // Almacenamiento de sesión de Redis
    │   │   ├── Session.php  // Clase de sesión
    │   │   └── mime.types   // Archivo de mapeo de mime
    │   ├── Http.php               // Implementación del protocolo http
    │   ├── Text.php               // Implementación del protocolo Text
    │   ├── Frame.php              // Implementación del protocolo Frame
    │   └── Websocket.php          // Implementación del protocolo websocket
    ├── Worker.php                 // Worker
    ├── WebServer.php              // Servidor web
    └── Autoloader.php             // Cargador automático de clases
```
