# Estrutura de diretório
```php
Workerman                        // núcleo do workerman
    ├── Connection                 // conexões de soquete relacionadas
    │   ├── ConnectionInterface.php // interface de conexão de soquete
    │   ├── TcpConnection.php       // classe de conexão Tcp
    │   ├── AsyncTcpConnection.php  // classe de conexão Tcp assíncrona
    │   └── UdpConnection.php       // classe de conexão Udp
    ├── Events                     // biblioteca de eventos de rede
    │   ├── EventInterface.php      // interface da biblioteca de eventos de rede
    │   ├── Event.php               // biblioteca de eventos Libevent
    │   ├── Ev.php                  // biblioteca de eventos Libev
    │   ├── Swoole.php              // biblioteca de eventos Swoole
    │   └── Select.php              // biblioteca de eventos Select
    ├── Lib                        // biblioteca de classes comuns
    │   ├── Constants.php           // definição de constantes
    │   └── Timer.php               // temporizador
    ├── Protocols                  // relacionado a protocolos
    │   ├── ProtocolInterface.php   // classe de interface de protocolo
    │   ├── Http                    // relacionado ao protocolo http
    │   │   ├── Chunk.php           // classe chunk http
    │   │   ├── Request.php         // classe de solicitação http
    │   │   ├── Response.php        // classe de resposta http
    │   │   ├── ServerSentEvents.php // classe de SSE
    │   │   ├── Session
    │   │   │   ├── FileSessionHandler.php // manipulador de sessão de arquivo
    │   │   │   └── RedisSessionHandler.php // manipulador de sessão Redis
    │   │   ├── Session.php         // classe de sessão
    │   │   └── mime.types          // arquivo de mapeamento de mime
    │   ├── Http.php                // implementação do protocolo http
    │   ├── Text.php                // implementação do protocolo de texto
    │   ├── Frame.php               // implementação do protocolo de frame
    │   └── Websocket.php           // implementação do protocolo websocket
    ├── Worker.php                 // Worker
    ├── WebServer.php              // WebServer
    └── Autoloader.php             // carregador automático de classes
```
