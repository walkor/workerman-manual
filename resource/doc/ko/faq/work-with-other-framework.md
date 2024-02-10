# 다른 프레임워크와의 통합 방법
**질문:**
다른 MVC 프레임워크 (예: thinkPHP, Yii 등)와 어떻게 통합하나요?

**답변:**

![workerman-thinkphp](../images/workerman-work-with-thinkphp.png)

다른 MVC 프레임워크와의 통합 **건의사항** (예: ThinkPHP를 기준으로):

1. ThinkPHP와 Workerman은 두 개의 독립된 시스템으로, 서로 다른 서버에 배포하여 서로 상호간섭하지 않도록 합니다.

2. ThinkPHP는 HTTP 프로토콜을 통해 웹 페이지를 브라우저에 렌더링하여 제공합니다.

3. ThinkPHP에서 제공하는 페이지의 JavaScript가 웹소켓 연결을 시작하고, Workerman에 연결합니다.

4. 연결 후 Workerman에 사용자가 속하는 웹소켓 연결을 인증할 데이터 패킷(사용자 이름 및 암호 또는 토큰 문자열)을 보냅니다.

5. 브라우저에 데이터를 푸시해야하는 경우에만 ThinkPHP에서 Workerman의 소켓 인터페이스를 호출하여 데이터를 푸시합니다.

6. 나머지 요청은 기존의 ThinkPHP의 HTTP 방식으로 처리됩니다.


**요약:**
Workerman을 브라우저로 데이터를 푸시할 수 있는 채널로 사용하고, 브라우저로 데이터를 푸시해야하는 경우에만 Workerman 인터페이스를 호출하여 푸시를 완료합니다. 비즈니스 로직은 모두 ThinkPHP에서 처리됩니다.


ThinkPHP에서 Workerman 소켓 인터페이스를 어떻게 호출하여 데이터를 푸시하는지는 [FAQ - 다른 프로젝트에서 푸시하는 방법](push-in-other-project.md) 섹션을 참조하십시오.

**ThinkPHP 공식으로 Workerman을 지원하고 있으며, [ThinkPHP5 매뉴얼](https://www.kancloud.cn/manual/thinkphp5/235128)을 참조하십시오.**
