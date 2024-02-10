# Dosya Yapısı
```plaintext
Workerman                      // workerman çekirdek kodu
    ├── Connection                 // socket bağlantısı ile ilgili
    │   ├── ConnectionInterface.php// socket bağlantısı arayüzü
    │   ├── TcpConnection.php      // Tcp bağlantı sınıfı
    │   ├── AsyncTcpConnection.php // Asenkron Tcp bağlantı sınıfı
    │   └── UdpConnection.php      // Udp bağlantı sınıfı
    ├── Events                     // Ağ olayları kütüphanesi
    │   ├── EventInterface.php     // Ağ olayları kütüphanesi arayüzü
    │   ├── Event.php              // Libevent ağ olayları kütüphanesi
    │   ├── Ev.php                 // Libev ağ olayları kütüphanesi
    │   ├── Swoole.php             // Swoole ağ olayları kütüphanesi
    │   └── Select.php             // Seçme ağ olayları kütüphanesi
    ├── Lib                        // Yaygın kütüphaneler
    │   ├── Constants.php          // Sabit tanımları
    │   └── Timer.php              // Zamanlayıcı
    ├── Protocols                  // Protokol ile ilgili
    │   ├── ProtocolInterface.php  // Protokol arayüz sınıfı
    │   ├── Http                   // http protokolü ile ilgili
    │   │   ├── Chunk.php    // http chunk sınıfı
    │   │   ├── Request.php  // http istek sınıfı
    │   │   ├── Response.php  // http cevap sınıfı
    │   │   ├── ServerSentEvents.php  // SSE sınıfı
    │   │   ├── Session
    │   │   │   ├── FileSessionHandler.php  // session dosya depolama
    │   │   │   └── RedisSessionHandler.php // session redis depolama
    │   │   ├── Session.php  // session sınıfı
    │   │   └── mime.types   // mime eşleme dosyası
    │   ├── Http.php               // http protokolü uygulaması
    │   ├── Text.php               // Metin protokolü uygulaması
    │   ├── Frame.php              // Frame protokolü uygulaması
    │   └── Websocket.php          // Websocket protokolü uygulaması
    ├── Worker.php                 // Worker
    ├── WebServer.php              // Web Sunucusu
    └── Autoloader.php             // Otomatik yükleme sınıfı
```
