# 환경 요구사항

## Windows 사용자
Workerman은 3.5.3 버전부터 Linux 및 Windows 시스템을 모두 지원합니다.

1. PHP >= 5.4 이상 및 PHP 환경 변수가 설정되어 있어야 합니다.
2. Windows 버전의 Workerman은 어떠한 확장도 의존하지 않습니다.
3. 설치 및 사용 제한은 [여기](https://www.workerman.net/windows)에서 확인하실 수 있습니다.
4. Windows에서 Workerman에는 여러 제한사항이 있으므로 정식 환경에서는 Linux 시스템을 권장하며, Windows 시스템은 개발 환경에만 권장됩니다.

``` ==== 이 페이지 이하 부분은 Linux 사용자에게만 해당하며, Windows 사용자는 무시하시기 바랍니다. ====```

## Linux 사용자 (Mac OS 포함)
Linux 사용자는 Linux 버전의 Workerman만 사용할 수 있습니다.

1. PHP >= 5.4를 설치하고 pcntl 및 posix 확장을 설치해야 합니다.
2. event 확장을 설치하는 것이 좋지만 필수는 아닙니다 (단, event 확장은 PHP >= 5.4 이상이 필요합니다).

### Linux 환경 검사 스크립트
Linux 사용자는 아래 스크립트를 실행하여 현재 환경이 Workerman 요구사항을 충족하는지 확인할 수 있습니다.

```curl -Ss https://www.workerman.net/check | php```

스크립트에서 모든 항목이 "ok"로 표시되면 Workerman 실행 환경을 충족하는 것입니다.

(참고: 검사 스크립트에서 event 확장을 확인하지는 않으며, 동시 접속 수가 1024 초과일 때에는 event 확장을 권장하지만, 설치 방법은 다음 섹션에서 확인할 수 있습니다.)

## 자세한 설명

### PHP-CLI에 관하여

Workerman은 [PHP Command Line Interface](https://php.net/manual/zh/features.commandline.php)(PHP-CLI) 모드로 실행됩니다. PHP-CLI는 PHP-FPM이나 Apache의 MOD-PHP와는 별개의 실행 가능한 프로그램으로, 서로 간의 충돌이나 의존 관계가 없는 완전히 독립적인 프로그램입니다.

### Workerman이 의존하는 확장에 관하여

1. [pcntl 확장](https://cn2.php.net/manual/zh/book.pcntl.php)

   pcntl 확장은 Linux 환경에서 프로세스 제어에 중요한 역할을 합니다. Workerman은 [프로세스 생성](https://cn2.php.net/manual/zh/function.pcntl-fork.php), [시그널 제어](https://cn2.php.net/manual/zh/function.pcntl-signal.php), [타이머](https://cn2.php.net/manual/zh/function.pcntl-alarm.php), [프로세스 상태 모니터링](https://cn2.php.net/manual/zh/function.pcntl-waitpid.php) 등의 기능을 사용합니다. 이 확장은 Windows 플랫폼에서 지원되지 않습니다.

2. [posix 확장](https://cn2.php.net/manual/zh/book.posix.php)

   posix 확장은 PHP가 시스템의 [POSIX 표준](https://baike.baidu.com/view/209573.htm) 인터페이스를 호출할 수 있도록합니다. Workerman은 주로 데몬화 및 사용자 그룹 제어 기능을 구현하는 데 사용되었습니다. 이 확장은 Windows 플랫폼에서 지원되지 않습니다.

3. [Event 확장](https://php.net/manual/zh/book.event.php) 또는 [libevent 확장](https://cn2.php.net/manual/en/book.libevent.php)

   event 확장을 통해 PHP는 [Epoll](https://baike.baidu.com/view/1385104.htm), Kqueue 등의 고급 이벤트 처리 메커니즘을 사용할 수 있으며, 이를 통해 Workerman은 고성능 및 고난이도의 연결 시 CPU 이용률을 크게 향상할 수 있습니다. 고난이도의 장기 연결 관련 응용프로그램에서 매우 중요합니다. libevent 확장(또는 event 확장)은 필수는 아니며, 미설치 시 PHP 기본 Select 이벤트 처리 메커니즘을 사용합니다.

## 확장 설치 방법

[확장 설치](../appendices/install-extension.md) 섹션을 참고하세요.
