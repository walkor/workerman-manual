# Workerman은 어떤 프로토콜을 지원하나요?

Workerman은 인터페이스에서 다양한 프로토콜을 지원하며, ```ConnectionInterface``` 인터페이스를 충족하면 됩니다(사용자 정의 통신 프로토콜 섹션 참조).

개발자 편의를 위해 Workerman은 HTTP 프로토콜, WebSocket 프로토콜 및 매우 간단한 Text 텍스트 프로토콜, 이진 전송에 사용할 수 있는 프레임 프로토콜을 제공합니다. 개발자는 이러한 프로토콜을 직접 사용할 수 있으며, 더 이상 이를 두 번 개발할 필요가 없습니다. 이러한 프로토콜 중 어느 것도 충족되지 않는 경우 개발자는 사용자 정의 프로토콜 섹션을 참조하여 자체 프로토콜을 구현할 수 있습니다.

또한 개발자는 직접적으로 tcp 또는 udp 프로토콜을 기반으로 할 수도 있습니다.

프로토콜 사용 예시
```php
// http 프로토콜
$worker1 = new Worker('http://0.0.0.0:1221');
// websocket 프로토콜
$worker2 = new Worker('websocket://0.0.0.0:1222');
// text 텍스트 프로토콜 (telnet 프로토콜)
$worker3 = new Worker('text://0.0.0.0:1223');
// frame 프레임 프로토콜 (이진 전송에 사용 가능)
$worker3 = new Worker('frame://0.0.0.0:1223');
// 직접적으로 tcp 전송
$worker4 = new Worker('tcp://0.0.0.0:1224');
// 직접적으로 udp 전송
$worker5 = new Worker('udp://0.0.0.0:1225');
```
