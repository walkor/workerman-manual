# Connection 클래스에서 제공하는 인터페이스

Workerman에는 두 가지 중요한 클래스 Worker와 Connection이 있습니다.

각 클라이언트 연결에는 하나의 Connection 객체가 대응하며, 이 객체의 onMessage, onClose 등의 콜백을 설정할 수 있습니다. 또한 클라이언트에 데이터를 전송하는 send 인터페이스와 연결을 닫는 close 인터페이스, 그리고 다른 필수적인 인터페이스가 제공됩니다.

Worker는 클라이언트 연결을 수신하고 연결을 개발자가 작업할 수 있도록 연결 객체로 래핑하는 리스닝 컨테이너 역할을 합니다.
