# 디렉토리 구조
```
Workerman                      // workerman 코어 코드
    ├── Connection                 // 소켓 연결 관련
    │   ├── ConnectionInterface.php// 소켓 연결 인터페이스
    │   ├── TcpConnection.php      // Tcp 연결 클래스
    │   ├── AsyncTcpConnection.php // 비동기 Tcp 연결 클래스
    │   └── UdpConnection.php      // Udp 연결 클래스
    ├── Events                     // 네트워크 이벤트 라이브러리
    │   ├── EventInterface.php     // 네트워크 이벤트 라이브러리 인터페이스
    │   ├── Event.php              // Libevent 네트워크 이벤트 라이브러리
    │   ├── Ev.php                 // Libev 네트워크 이벤트 라이브러리
    │   ├── Swoole.php             // Swoole 네트워크 이벤트 라이브러리
    │   └── Select.php             // Select 네트워크 이벤트 라이브러리
    ├── Lib                        // 일반적으로 사용되는 라이브러리
    │   ├── Constants.php          // 상수 정의
    │   └── Timer.php              // 타이머
    ├── Protocols                  // 프로토콜 관련
    │   ├── ProtocolInterface.php  // 프로토콜 인터페이스 클래스
    │   ├── Http                   // http 프로토콜 관련
    │   │   ├── Chunk.php    // http chunk 클래스
    │   │   ├── Request.php  // http 요청 클래스
    │   │   ├── Response.php  // http 응답 클래스
    │   │   ├── ServerSentEvents.php  // SSE 클래스
    │   │   ├── Session
    │   │   │   ├── FileSessionHandler.php  // 세션 파일 저장소
    │   │   │   └── RedisSessionHandler.php // 세션 Redis 저장소
    │   │   ├── Session.php  // 세션 클래스
    │   │   └── mime.types   // mime 매핑 파일
    │   ├── Http.php               // http 프로토콜 구현
    │   ├── Text.php               // Text 프로토콜 구현
    │   ├── Frame.php              // Frame 프로토콜 구현
    │   └── Websocket.php          // 웹소켓 프로토콜 구현
    ├── Worker.php                 // Worker
    ├── WebServer.php              // WebServer
    └── Autoloader.php             // 자동으로 클래스 로드
```
