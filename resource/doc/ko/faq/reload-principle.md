# 부드러운 다시 시작 원리
## 부드러운 다시 시작이란?

부드러운 다시 시작은 일반적인 다시 시작과 다르며, 짧은 연결 비즈니스(일반적으로 단일 연결 비즈니스를 의미)에 영향을 주지 않고 서비스를 다시 시작할 수 있게 하여 PHP 프로그램을 다시로드하고 비즈니스 코드 업데이트를 완료할 수 있습니다.

부드러운 다시 시작은 일반적으로 비즈니스 업데이트 또는 버전 배포 과정에서 적용되며, 코드 배포로 인한 일시적인 서비스 중단을 방지할 수 있습니다.

> **주의**
> Windows 시스템은 다시로드를 지원하지 않습니다.

> **주의**
> 장기 연결(예: 웹 소켓) 비즈니스의 경우, 프로세스에 부드러운 다시 시작시 연결이 끊길 수 있습니다. 해결 방법은 [gatewayWorker](https://www.workerman.net/doc/gateway-worker)와 같은 아키텍처를 사용하여 연결을 유지하고,이 그룹의 프로세스의 [reloadable](../worker/reloadable.md) 속성을 false로 설정하는 것입니다. 비즈니스 로직은 다른 worker 프로세스를 시작하여 처리하고, gateway와 worker 프로세스는 tcp 통신을 통해 상호 호출합니다. 비즈니스를 변경해야 할 때는 worker 프로세스 만 다시 시작하면 됩니다.

## 제한
**참고 : on {...} 콜백에서 로드된 파일만 부드러운 다시 시작 후 자동으로 업데이트됩니다. 시작 스크립트에서 직접 로드하는 파일 또는 하드 코딩된 코드는 재로드로 자동으로 업데이트되지 않습니다.**

#### 다음 코드는 다시로드 후 업데이트되지 않습니다.
```php
$worker = new Worker('http://0.0.0.0:1234');
$worker->onMessage = function($connection, $request) {
    $connection->send('hi'); // 하드 코딩된 코드는 업데이트를 지원하지 않습니다.
};
```

```php
$worker = new Worker('http://0.0.0.0:1234');
require_once __DIR__ . '/your/path/MessageHandler.php'; // 시작 스크립트에서 직접로드된 파일은 업데이트를 지원하지 않습니다.
$messageHandler = new MessageHandler();
$worker->onMessage = [$messageHandler, 'onMessage']; // 가정: MessageHandler 클래스에 onMessage 메서드가 있다고 가정합니다.
```


#### 다음 코드는 다시로드 후 자동으로 업데이트됩니다.
```php
$worker = new Worker('http://0.0.0.0:1234');
$worker->onWorkerStart = function($worker) { // onWorkerStart는 프로세스 시작 후 트리거됩니다.
    require_once __DIR__ . '/your/path/MessageHandler.php'; // 프로세스 시작 후에 로드된 파일은 업데이트를 지원합니다.
    $messageHandler = new MessageHandler();
    $worker->onMessage = [$messageHandler, 'onMessage'];
};
```
MessageHandler.php를 변경한 후 `php start.php reload`를 실행하면, MessageHandler.php가 메모리에 다시로드되어 비즈니스 로직이 업데이트됩니다.

> **팁**
> 위의 코드는 편의를 위해 `require_once` 문을 사용했지만, 프로젝트가 psr4 자동로드를 지원하는 경우에는 `require_once` 문을 호출할 필요가 없습니다.

## 부드러운 다시 시작의 원리

Workerman은 주 프로세스와 자식 프로세스로 나뉘며, 주 프로세스는 자식 프로세스를 모니터링하고, 자식 프로세스는 클라이언트의 연결을 받아들이고 요청 데이터를 처리하여 클라이언트에 응답합니다. 비즈니스 코드가 업데이트되면 사실상 자식 프로세스만 업데이트하면 코드가 업데이트됩니다.

Workerman 주 프로세스가 부드러운 다시 시작 신호를받으면, 주 프로세스는 하나의 자식 프로세스에 안전한 종료 (해당 프로세스가 현재 요청을 처리한 후 종료) 신호를 보냅니다. 이 프로세스가 종료되면, 주 프로세스는 새로운 PHP 코드를로드 한 새로운 자식 프로세스를 (이 자식 프로세스) 다시 만들고, 주 프로세스는 다시 다른 이전 프로세스에 중지 명령을 보내어 이전 프로세스가 모두 교체 될 때까지 한 프로세스씩 재시작합니다.

우리는 부드러운 다시 시작이 사실상 예전의 비즈니스 프로세스를 하나씩 종료하고 새로운 프로세스를 순차적으로 생성하여 업데이트하는 것을 볼 수 있습니다. 부드러운 다시 시작 시 사용자에게 영향을 미치지 않기 위해, 프로세스에 사용자와 관련된 상태 정보를 저장하지 않는 것이 좋으며, 즉 비즈니스 프로세스가 상태를 유지하지 않도록하여 프로세스가 종료되어도 정보가 손실되지 않도록 하는 것이 필요합니다.
