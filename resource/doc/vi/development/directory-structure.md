# Cấu trúc thư mục
```plaintext
Workerman                      // mã nguồn lõi của Workerman
    ├── Connection                 // liên kết socket liên quan
    │   ├── ConnectionInterface.php// giao diện liên kết socket
    │   ├── TcpConnection.php      // lớp kết nối Tcp
    │   ├── AsyncTcpConnection.php // lớp kết nối Tcp không đồng bộ
    │   └── UdpConnection.php      // lớp kết nối Udp
    ├── Events                     // thư viện sự kiện mạng
    │   ├── EventInterface.php     // giao diện thư viện sự kiện mạng
    │   ├── Event.php              // thư viện sự kiện mạng Libevent
    │   ├── Ev.php                 // thư viện sự kiện mạng Libev
    │   ├── Swoole.php             // thư viện sự kiện mạng Swoole
    │   └── Select.php             // thư viện sự kiện mạng Select
    ├── Lib                        // thư viện lớp thông dụng
    │   ├── Constants.php          // định nghĩa hằng số
    │   └── Timer.php              // bộ định thời
    ├── Protocols                  // liên quan đến giao thức
    │   ├── ProtocolInterface.php  // lớp giao thức giao diện
    │   ├── Http                   // liên quan đến giao thức http
    │   │   ├── Chunk.php    // lớp chunk http
    │   │   ├── Request.php  // lớp yêu cầu http
    │   │   ├── Response.php  // lớp phản hồi http
    │   │   ├── ServerSentEvents.php  // lớp SSE
    │   │   ├── Session
    │   │   │   ├── FileSessionHandler.php  // lớp xử lý phiên lưu trữ tệp
    │   │   │   └── RedisSessionHandler.php // lớp xử lý phiên lưu trữ redis
    │   │   ├── Session.php  // lớp phiên
    │   │   └── mime.types   // tệp ánh xạ mime
    │   ├── Http.php               // thực hiện giao thức http
    │   ├── Text.php               // thực hiện giao thức Text
    │   ├── Frame.php              // thực hiện giao thức Frame
    │   └── Websocket.php          // thực hiện giao thức websocket
    ├── Worker.php                 // Worker
    ├── WebServer.php              // WebServer
    └── Autoloader.php             // lớp tự động tải
```
