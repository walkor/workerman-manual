# Структура каталога
```
Workerman                      // ядро Workerman
    ├── Connection                 // связь по сокетам
    │   ├── ConnectionInterface.php// интерфейс связи по сокетам
    │   ├── TcpConnection.php      // класс Tcp-соединения
    │   ├── AsyncTcpConnection.php // класс асинхронного Tcp-соединения
    │   └── UdpConnection.php      // класс Udp-соединения
    ├── Events                     // библиотека сетевых событий
    │   ├── EventInterface.php     // интерфейс библиотеки сетевых событий
    │   ├── Event.php              // библиотека сетевых событий Libevent
    │   ├── Ev.php                 // библиотека сетевых событий Libev
    │   ├── Swoole.php             // библиотека сетевых событий Swoole
    │   └── Select.php             // библиотека сетевых событий Select
    ├── Lib                        // общие библиотеки
    │   ├── Constants.php          // определение констант
    │   └── Timer.php              // таймер
    ├── Protocols                  // связанные с протоколами
    │   ├── ProtocolInterface.php  // интерфейс класса протокола
    │   ├── Http                   // связанные с протоколом HTTP
    │   │   ├── Chunk.php    // класс http-чанка
    │   │   ├── Request.php  // класс запроса HTTP
    │   │   ├── Response.php  // класс ответа HTTP
    │   │   ├── ServerSentEvents.php  // класс SSE
    │   │   ├── Session
    │   │   │   ├── FileSessionHandler.php  // обработчик файлов для сессий
    │   │   │   └── RedisSessionHandler.php // обработчик Redis для сессий
    │   │   ├── Session.php  // класс сессии
    │   │   └── mime.types   // файл карты соответствия mime-типов
    │   ├── Http.php               // реализация протокола HTTP
    │   ├── Text.php               // реализация протокола Text
    │   ├── Frame.php              // реализация протокола Frame
    │   └── Websocket.php          // реализация протокола WebSocket
    ├── Worker.php                 // Worker
    ├── WebServer.php              // Веб-сервер
    └── Autoloader.php             // автозагрузчик классов
```
