# 함수 비활성화 확인

이 스크립트를 사용하여 비활성화된 함수를 확인합니다. 명령 라인에서 `curl -Ss https://www.workerman.net/check | php`을 실행합니다.

`function 함수명 may be disabled. Please check disable_functions in php.ini`과 같은 메시지가 나오면 workerman이 의존하는 함수가 비활성화되어 있으며, 이를 해제하지 않으면 workerman을 정상적으로 사용할 수 없습니다. php.ini에서 비활성화를 해제하려면 다음 두 가지 방법 중 하나를 선택하십시오.

## 방법 1: 스크립트로 해제

`curl -Ss https://www.workerman.net/fix | php` 스크립트를 실행하여 비활성화를 해제합니다.

## 방법 2: 수동 해제

**다음 단계를 따라 진행하세요:**

1. `php --ini`를 실행하여 php cli에서 사용하는 php.ini 파일의 위치를 찾습니다.

2. php.ini를 열고 `disable_functions` 항목을 찾아 해당 함수의 비활성화를 해제합니다.

**의존하는 함수**
workerman을 사용하려면 다음 함수의 비활성화를 해제해야 합니다.
```
stream_socket_server
stream_socket_client
pcntl_signal_dispatch
pcntl_signal
pcntl_alarm
pcntl_fork
posix_getuid
posix_getpwuid
posix_kill
posix_setsid
posix_getpid
posix_getpwnam
posix_getgrnam
posix_getgid
posix_setgid
posix_initgroups
posix_setuid
posix_isatty
```
