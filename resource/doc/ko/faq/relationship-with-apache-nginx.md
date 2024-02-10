# Apache 및 Nginx와의 관계
**질문:**
Workerman과 Apache/nginx/php-fpm의 관계는 무엇인가요? Workerman과 Apache/nginx/php-fpm은 충돌하나요?

**대답:**
Workerman과 Apache/nginx/php-fpm은 아무런 관련이 없으며, Workerman은 Apache/nginx/php-fpm에 의존하지 않습니다. 이들은 모두 독립적인 컨테이너로, 서로 간섭하지 않으며 (동일한 포트를 수신하지 않는 한) 충돌하지 않습니다.

Workerman은 일반적인 소켓 서버 프레임워크로, 지속적인 연결을 지원하며 HTTP, WebSocket 및 사용자 정의 프로토콜과 같은 다양한 프로토콜을 지원합니다. 반면에 Apache/nginx/php-fpm은 일반적으로 HTTP 프로토콜의 웹 프로젝트를 개발하는 데 사용됩니다.

서버가 이미 Apache/nginx/php-fpm으로 구축되어 있다면, Workerman을 추가로 구축해도 그들의 작동에 영향을 미치지 않습니다.
