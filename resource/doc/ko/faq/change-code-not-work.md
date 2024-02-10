# 코드를 변경한 후에 적용되지 않는 이유

**이유:**

Workerman은 계속해서 메모리에 상주하여 디스크의 반복적인 읽기와 PHP의 반복적인 해석 및 컴파일을 피해 최고의 성능을 달성할 수 있는 상태입니다. 따라서 비즈니스 코드를 변경한 후에는 수동으로 reload 또는 restart를 해야만 적용됩니다.

동시에 Workerman은 파일 업데이트를 감지하는 서비스를 제공하며, 해당 서비스는 파일이 업데이트됨을 감지하면 자동으로 reload하여 PHP 파일을 다시 로드합니다. 개발자는 프로젝트와 함께 해당 서비스를 추가하여 프로젝트를 시작할 때 함께 실행할 수 있습니다.

참고: Windows 시스템에서는 reload를 지원하지 않으므로 감시 서비스를 사용할 수 없습니다.

**파일 감시 서비스 다운로드 링크:**

1. 의존성 없는 버전: https://github.com/walkor/workerman-filemonitor
2. inotify에 의존하는 버전: https://github.com/walkor/workerman-filemonitor-inotify

**두 버전의 차이:**

Link 1 버전은 파일 업데이트 시간을 매 초마다 폴링하여 파일이 업데이트되었는지 확인하는 방법을 사용합니다.

Link 2는 Linux 커널 내 [inotify](https://baike.baidu.com/view/2645027.htm) 메커니즘을 활용하여 파일이 업데이트되면 시스템이 Workerman에게 알림을 보냅니다.

일반적으로 의존성 없는 Link 1 버전을 사용하는 것이 충분합니다.
