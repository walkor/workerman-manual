# 서문

**Workerman, 고성능 PHP 어플리케이션 컨테이너**

## Workerman이란 무엇인가?
Workerman은 순수한 PHP로 개발된 고성능의 오픈 소스 PHP 어플리케이션 컨테이너입니다.

Workerman은 바퀴를 재발명하는 것이 아니며, MVC 프레임워크가 아니라 더 깊은 수준의 보다 일반적인 서비스 프레임워크입니다. Workerman은 TCP 프록시, 프록시 서버, 게임 서버, 메일 서버, FTP 서버, 심지어 PHP 버전의 Redis, PHP 버전의 데이터베이스, PHP 버전의 Nginx, PHP 버전의 PHP-FPM등을 개발하는 데 사용할 수 있습니다. Workerman은 PHP 분야의 혁신으로 개발자들이 PHP로 웹 개발만 하는 것을 완전히 벗어나게 했습니다.

실제로 Workerman은 PHP 버전의 Nginx와 유사하며, 핵심은 멀티 프로세스 + epoll + 비차단 IO입니다. Workerman은 각 프로세스마다 수만 개의 동시 연결을 유지할 수 있습니다. 메모리에 상주하므로 Apache, Nginx, PHP-FPM과 같은 컨테이너에 의존하지 않고 매우 높은 성능을 제공합니다. 또한 TCP, UDP, UNIXSOCKET를 지원하며, 장기간 연결을 지원하며, WebSocket, HTTP, WSS, HTTPS 등의 통신 프로토콜 및 여러 사용자 정의 프로토콜을 지원합니다. 타이머, 비동기 소켓 클라이언트, 비동기 Redis, 비동기 HTTP, 비동기 메시지 큐 등 다양한 고성능 컴포넌트를 제공합니다.

## Workerman의 일부 응용 분야
Workerman은 전통적인 MVC 프레임워크와는 다르게 웹 개발뿐만 아니라 실시간 통신, 사물 인터넷, 게임, 서비스 관리, 기타 서버 또는 미들웨어와 같은 더 광범위한 응용 분야에 사용될 수 있습니다. 이를 통해 PHP 개발자의 시야를 크게 확대시켜줍니다. 현재 이러한 분야의 PHP 개발자는 매우 부족하며, PHP 분야에서 기술적 우위를 가지고 싶다면 매일의 CRUD 작업에 만족하지 않거나 아키텍트 또는 기술 전문가로 발전하고 싶다면 Workerman은 매우 학습 가치 있는 프레임워크입니다. 개발자들은 Workerman을 사용하는 것뿐만 아니라, 자신만의 오픈 소스 프로젝트를 기반으로 발전시켜 기술력을 향상시키고 영향력을 확대시킬 수 있습니다. [Beanbun 다중 프로세스 네트워크 크롤러 프레임워크](https://github.com/kiddyuchina/Beanbun)가 이를 잘 보여주는 사례입니다.

Workerman의 일부 응용 분야는 다음과 같습니다:

1. 실시간 통신
   - 웹에서의 실시간 채팅, 실시간 메시지 푸시, WeChat Mini Program, 모바일 앱 메시지 푸시, PC 소프트웨어 메시지 푸시 등
   [[예시: workerman-chat 채팅방](https://www.workerman.net/workerman-chat), [웹 메시지 푸시](https://www.workerman.net/web-sender), [작은 올챙이 채팅방](https://www.workerman.net/workerman-todpole)]

2. 사물 인터넷
   - Workerman과 프린터 통신, 마이크로컨트롤러 통신, 스마트 워치, 스마트 홈, 공유 자전거 등
   [고객 사례로는 이연클라우드, EasyPark Era 등]

3. 게임 서버
   - 보드 게임, MMORPG 게임 등
   [[예시: browserquest-php](https://www.workerman.net/browserquest)]

4. HTTP 서비스
   - 고성능 HTTP 인터페이스, 고성능 웹사이트 개발. HTTP 관련 서비스 또는 사이트를 구축하려면 [webman](https://github.com/walkor/webman)을 강력히 추천합니다.

5. SOA 서비스화
   - Workerman을 사용하여 기존 비즈니스 기능 단위를 서비스 형태로 외부에 통합하여 시스템을 유연하게 만들고 유지보수하기 쉽고 높은 가용성과 탄력성을 제공합니다. [[예시: workerman-json-rpc](https://github.com/walkor/workerman-jsonrpc), [workerman-thrift](https://github.com/walkor/workerman-thrift)]

6. 기타 서버 소프트웨어
   - [GatewayWorker](https://www.workerman.net/doc/gateway-worker), [PHPSocket.IO](https://www.workerman.net/phpsocket_io), [HTTP 프록시](https://github.com/walkor/php-http-proxy), [sock5 프록시](https://github.com/walkor/php-socks5), [분산 통신 컴포넌트](https://github.com/walkor/Channel), [분산 변수 공유 컴포넌트](https://github.com/walkor/GlobalData), [메시지 큐](https://github.com/walkor/workerman-queue), DNS 서버, 웹 서버, CDN 서버, FTP 서버 등

7. 컴포넌트
   - [비동기 Redis](components/workerman-redis.md), [비동기 HTTP 클라이언트](components/workerman-http-client.md), 사물 인터넷 MQTT 클라이언트](components/workerman-mqtt.md), 메시지 큐 [workerman/redis-queue](components/workerman-redis-queue.md), [workerman/stomp](components/workerman-stomp.md), [workerman/rabbitmq](components/workerman-rabbitmq.md), [파일 모니터링 컴포넌트](components/file-monitor.md) 등 다양한 타사 개발 컴포넌트 프레임워크들

전통적인 MVC 프레임워크로는 위의 기능들을 구현하는 것이 매우 어렵기 때문에 Workerman이 탄생한 이유입니다.

## Workerman의 철학
극도로 간결, 안정적, 고성능, 분산형.

### **극도로 간결**
작으면 아름답다. Workerman 커널은 간결하며 몇 개의 PHP 파일만을 노출시키며, 학습하기 매우 간단합니다. 모든 다른 기능은 컴포넌트 방식으로 확장됩니다.

Workerman은 완벽한 문서 + 권위있는 홈페이지 + 활발한 커뮤니티 + 여러 고성능 컴포넌트 + 다양한 예제를 가지고 있어 개발자들이 쉽게 사용할 수 있도록 합니다.

### **안정적**
Workerman은 이미 몇 년간 오픈 소스로 공개되어 많은 상장회사들이 대규모로 사용하고 있으며, 매우 안정적입니다. 몇 년 동안 재부팅되지 않은 서비스도 여전히 빠르게 운영 중입니다. Core dump, 메모리 누수, 버그 없음.

### **고성능**
Workerman은 메모리 상주 때문에 Apache, Nginx, PHP-FPM에 의존하지 않으며, 요청마다 모든 것을 초기화하고 소멸하는 오버헤드가 없으므로 전통적인 MVC 프레임워크보다 수십 배 높은 성능을 제공합니다. PHP7에서 ab(stress test tool) 테스트로 QPS가 독립적인 Nginx보다도 높습니다.

### **분산형**
이제는 혼자 무장한 시대가 아닙니다. 단일 서버의 성능이 어떻게 강력하더라도 분산형 다중 서버 배포가 핵심입니다. Workerman은 [GatewayWorker 프레임워크](https://doc2.workerman.net)를 통해 분산형 장기 연결 통신 솔루션을 직접 제공하며, 단순한 구성 및 시작으로 서버 성능을 두 배로 증가시킬 수 있습니다. TCP 장기 연결 애플리케이션을 개발하려면 [GatewayWorker](https://doc2.workerman.net)를 직접 사용하는 것이 좋으며, 이것은 Workerman의 래핑으로 장기 연결 애플리케이션에 더 풍부한 인터페이스와 분산 처리 기능을 제공합니다.

## 이 매뉴얼의 범위
Workerman 3.x - 4.x 버전

## Windows 사용자 (필독)
Workerman은 Linux 및 Windows 시스템을 모두 지원합니다. Windows 버전의 Workerman **는 어떤 확장도 의존하지 않으며**, PHP 환경 변수를 설정하기만 하면 됩니다. **Windows 버전의 Workerman 설치 및 유의 사항은 [Windows 사용자를 위한 필독사항](https://www.workerman.net/windows)을 참조하십시오.**

## 클라이언트
WorkerMan의 통신 프로토콜은 개방적이며 맞춤적이므로 이론적으로 WorkerMan은 어떤 플랫폼의 클라이언트와도 통신할 수 있습니다. 사용자는 클라이언트를 개발할 때 해당 통신 프로토콜에 따라 서버와 통신을 완료할 수 있습니다.
