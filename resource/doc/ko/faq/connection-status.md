현재 클라이언트 연결 수를 확인하려면 ```php start.php status```를 실행하면 현재 서버의 Workerman이 실행되는 상태를 볼 수 있습니다. ```연결``` 필드는 각 프로세스의 현재 TCP 연결 수를 표시합니다. 이 필드는 클라이언트의 TCP 연결 수 뿐만 아니라 Workerman 내부 통신의 TCP 연결 수도 포함됩니다. 예를 들어 Workerman의 게이트웨이/워커 모델에서 각 게이트웨이 프로세스의 현재 클라이언트 연결 수는 ```connections``` 필드의 값에서 워커 프로세스 수를 뺀 값입니다.