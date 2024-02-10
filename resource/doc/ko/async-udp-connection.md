# AsyncUdpConnection

**(workerman 버전 3.0.8 이상이 필요합니다.)**

AsyncUdpConnection은 원격 UDP 서버와 통신하기 위한 UDP 클라이언트로 사용될 수 있습니다.

사실 UDP는 연결이 없는 프로토콜이지만, 여기서는 AsyncTcpConnection과의 일관성을 유지하기 위해 명명 규칙과 인터페이스를 기본적으로 동일하게 유지합니다.

**참고: AsyncTcpConnection과 달리, AsyncUdpConnection은 다음과 같은 속성 또는 메서드를 지원하지 않습니다.**
1. connection->id 속성이 없음
2. connection->worker 속성이 없음
3. connection->transport 속성이 없음
4. connection->maxSendBufferSize 속성이 없음
5. connection->defaultMaxSendBufferSize 속성이 없음
6. connection->maxPackageSize 속성이 없음
7. connection->onBufferFull 콜백이 없음
8. connection->onBufferDrain 콜백이 없음
9. connection->onError 콜백이 없음
10. connection->destroy() 메서드가 없음
11. connection->pauseRecv() 메서드가 없음
12. connection->resumeRecv() 메서드가 없음
13. connection->pipe() 메서드가 없음
14. connection->reconnect() 메서드가 없음

**AsyncUdpConnection이 지원하는 속성 또는 메서드**
1. connection->protocol 속성을 지원함
2. connection->onMessage 콜백을 지원함
3. connection->connect() 메서드를 지원함
4. connection->send() 메서드를 지원함
5. connection->getRemoteIp() 메서드를 지원함
6. connection->getRemotePort() 메서드를 지원함
7. connection->onClose 콜백을 지원함.
참고: TCP는 연결 기반이므로 일반적으로 양쪽 중 하나가 close를 호출하여 연결을 끊을 때 양쪽 모두 onClose를 트리거할 수 있습니다. 그러나 UDP는 연결이 없기 때문에 connection->close() 메서드를 호출하면 로컬 onClose 콜백만 트리거되고 상대편의 onClose 콜백은 트리거되지 않습니다.
