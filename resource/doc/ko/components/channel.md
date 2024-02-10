# Channel 분산 통신 구성 요소
**``` (Workerman 버전 >= 3.3.0 이상 필요) ```**

소스 코드 주소: https://github.com/walkor/Channel

Channel은 프로세스 간 통신 또는 서버 간 통신을 완료하기 위한 분산 통신 구성 요소입니다.

## 특징
1. 구독 및 발행 모델을 기반으로 함
2. 논블로킹 IO

## 원리
Channel은 Channel/Server 서버 및 Channel/Client 클라이언트로 구성됨

Channel/Client는 connect 인터페이스를 통해 Channel/Server에 연결하고 장기간 연결을 유지함

Channel/Client는 on 인터페이스를 호출하여 자신이 어떤 이벤트에 관심이 있는지 Channel/Server에 알리고 이벤트 콜백 함수를 등록함 (콜백은 Channel/Client의 프로세스에서 발생함)

Channel/Client는 publish 인터페이스를 사용하여 특정 이벤트 및 이벤트 관련 데이터를 Channel/Server에 발행함

Channel/Server는 이벤트 및 데이터를 받은 후 해당 이벤트에 관심이 있는 Channel/Client에게 분배함

Channel/Client는 이벤트 및 데이터를 받은 후 on 인터페이스에 설정된 콜백을 트리거함

Channel/Client는 자신이 관심을 갖고 있는 이벤트만 받고 콜백을 트리거함


## 설치

`composer require workerman/channel`

## 주의
Channel은 workerman 환경에서만 사용할 수 있으며 php-fpm 환경에서는 사용할 수 없습니다.
