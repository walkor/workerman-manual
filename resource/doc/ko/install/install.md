WorkerMan은 사실 PHP 코드 패키지이며, PHP 환경이 이미 설정되어 있다면 WorkerMan 소스 코드나 데모를 다운로드하여 실행할 수 있습니다.

**Composer로 설치:**
```sh
composer require workerman/workerman
```

> **주의**
> 일부 composer 프록시 미러가 완전하지 않을 수 있으므로 위 명령어를 사용하여 `composer config -g --unset repos.packagist`를 실행하여 프록시를 제거합니다.

# Windows 사용자 (필독)

Workerman 3.5.3 버전부터 Windows와 Linux 시스템을 동시에 지원합니다.
Windows 사용자는 PHP 환경 변수를 설정해야 합니다.

 ` ===이 페이지 이하는 리눅스 환경에서만 적용되며, Windows 사용자는 무시하십시오=== `

# 리눅스 시스템 환경 검사
리눅스 시스템에서 다음 스크립트를 사용하여 현재 PHP 환경이 WorkerMan 실행 요구 사항을 충족하는지를 테스트할 수 있습니다.
 `curl -Ss https://www.workerman.net/check | php`

위 스크립트가 모두 "OK"로 표시되면 WorkerMan 요구 사항을 충족한다는 것을 의미하며, [공식 웹 사이트](https://www.workerman.net/)에서 예제를 다운로드하여 실행할 수 있습니다.

모두 "OK"로 나오지 않으면 아래 문서를 참고하여 누락된 확장 기능을 설치하면 됩니다.

(참고: 검사 스크립트는 event 확장을 검사하지 않습니다. 업무 동시 연결 수가 1024보다 크면 event 확장을 설치해야 하며, [리눅스 커널 최적화](../appendices/kernel-optimization.md)를 해야 합니다. 확장 설치 방법은 아래 설명을 참조하십시오.)

# 기존 PHP 환경에 누락된 확장 설치

## pcntl 및 posix 확장 설치:

**CentOS 시스템**
php가 yum을 통해 설치된 경우, 명령줄에서 `yum install php-process`를 실행하여 pcntl 및 posix 확장을 설치할 수 있습니다.

만약 설치에 실패하거나 php 자체가 yum을 통해 설치되지 않은 경우에는 [부록-확장 설치](../appendices/install-extension.md) 섹션의 세 번째 방법인 소스 코드 컴파일 방법을 참고하십시오.

**Debian/Ubuntu/Mac OS 시스템**
[부록-확장 설치](../appendices/install-extension.md) 섹션을 참고하여 소스 코드 컴파일 방법을 사용하십시오.

## event 확장 설치:
더 많은 동시 연결을 지원하기 위해 event 확장을 반드시 설치해야 합니다. 또한 [리눅스 커널 최적화](../appendices/kernel-optimization.md)를 해야 합니다. 아래와 같이 설치합니다.

**CentOS 시스템**

1. event 확장에 필요한 libevent-devel 패키지를 설치하려면 명령줄에서 다음을 실행하십시오.
```shell
yum install libevent-devel -y
# 설치에 실패하는 경우 아래 명령어를 사용해 보십시오
# yum install libevent2-devel -y
``` 

2. event 확장을 설치하려면 명령줄에서 다음을 실행하십시오.
(event 확장은 PHP>=5.4를 요구합니다)
```shell
pecl install event
```
알림: `Include libevent OpenSSL support [yes] :`에서 `no`를 입력하고 Enter를 누르십시오. 기타 설정은 그냥 Enter를 누르면 됩니다.

3. `php --ini`를 실행하여 php.ini 파일을 찾아 열고 마지막 줄에 다음 설정을 추가하십시오.
```shell
extension=event.so
```

**Debian/Ubuntu 시스템**

1. event 확장에 필요한 libevent-dev 패키지를 설치하려면 명령줄에서 다음을 실행하십시오.
```shell
apt-get install libevent-dev -y
# 설치에 실패하는 경우 아래 명령어를 사용해 보십시오
# apt-get install libevent2-dev -y
```

2. event 확장을 설치하려면 명령줄에서 다음을 실행하십시오.
```shell
pecl install event
```
알림: `Include libevent OpenSSL support [yes] :`에서 `no`를 입력하고 Enter를 누르십시오. 기타 설정은 그냥 Enter를 누르면 됩니다.

3. `php --ini`를 실행하여 php.ini 파일을 찾아 열고 마지막 줄에 다음 설정을 추가하십시오.
```shell
extension=event.so
```

**Mac OS 시스템**

Mac 시스템은 주로 개발용으로 사용되기 때문에 event 확장을 설치할 필요가 없습니다.

# 새로운 시스템 설치 (PHP+확장 프로그램 신규 설치)

## CentOS 시스템 설치 안내

1. 명령줄에서 PHP CLI 프로그램, pcntl, posix, libevent 라이브러리 및 git을 설치하려면 다음을 실행하십시오.
```shell
yum install php-cli php-process git gcc php-devel php-pear libevent-devel -y
``` 

2. event 확장을 설치하려면 명령줄에서 다음을 실행하십시오.
(참고: event 확장은 PHP>=5.4를 요구합니다)
```shell
pecl install event
```
알림: `Include libevent OpenSSL support [yes] :`에서 `no`를 입력하고 Enter를 누르십시오. 기타 설정은 그냥 Enter를 누르면 됩니다.

3. `php --ini`를 실행하여 php.ini 파일을 찾아 열고 마지막 줄에 다음 설정을 추가하십시오.
```shell
extension=event.so
```

4. 명령줄에서 GitHub에서 WorkerMan 메인 프로그램을 다운로드하려면 다음을 실행하십시오.
```shell
git clone https://github.com/walkor/Workerman
```

5. [입문 가이드-간단한 개발 예제 섹션](../getting-started/simple-example.md)을 참고하여 진입 파일을 작성하고 실행하십시오. 아니면 [공식 웹 사이트](https://www.workerman.net/)에서 데모를 다운로드하여 실행하십시오.

## Debian/Ubuntu 시스템 설치 안내

1. 명령줄에서 PHP CLI 프로그램, libevent 및 git을 설치하려면 다음을 실행하십시오.
```shell
apt-get install php-cli git gcc php-pear php-dev libevent-dev -y
```

2. event 확장을 설치하려면 명령줄에서 다음을 실행하십시오.
(참고: event 확장은 PHP>=5.4를 요구합니다)
```shell
pecl install event
```
알림: `Include libevent OpenSSL support [yes] :`에서 `no`를 입력하고 Enter를 누르십시오. 기타 설정은 그냥 Enter를 누르면 됩니다.

3. `php --ini`를 실행하여 php.ini 파일을 찾아 열고 마지막 줄에 다음 설정을 추가하십시오.
```shell
extension=event.so
```

4. 명령줄에서 GitHub에서 WorkerMan 메인 프로그램을 다운로드하려면 다음을 실행하십시오.
```shell
git clone https://github.com/walkor/Workerman
```

5. [입문 가이드-간단한 개발 예제 섹션](../getting-started/simple-example.md)을 참고하여 진입 파일을 작성하고 실행하십시오. 아니면 [공식 웹 사이트](https://www.workerman.net/)에서 데모를 다운로드하여 실행하십시오.

## Mac OS 시스템 설치 안내
**방법 1:** Mac 시스템은 PHP CLI를 기본으로 제공하나 ```pcntl``` 확장이 부족할 수 있습니다.

1. [부록-확장 설치](../appendices/install-extension.md) 섹션의 세 번째 방법인 소스 코드 컴파일로 ```pcntl``` 확장을 설치하십시오.

2. [부록-확장 설치](../appendices/install-extension.md) 섹션의 네 번째 방법인 phpize를 사용하여 ```event``` 확장을 설치하십시오 (개발 시스템에서는 이 단계를 생략할 수 있음).

3. https://www.workerman.net/download/workermanzip에서 WorkerMan 메인 프로그램을 다운로드하거나 [공식 웹 사이트](https://www.workerman.net/)에서 데모를 다운로드하여 실행하십시오.

**방법 2:** ```brew``` 명령어를 사용하여 php 및 해당 확장을 설치하십시오.

1. 아래 명령어를 사용하여 ```brew``` 도구를 설치하십시오 (이미 설치되어 있는 경우 이 단계를 생략할 수 있습니다)
```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

2. 아래 명령어를 사용하여 ```php```를 설치하십시오.
```shell
brew install php
```

3. 아래 명령어를 사용하여 ```event``` 확장을 설치하십시오.
```shell
brew install php-event    
```

4. [공식 웹 사이트](https://www.workerman.net/)에서 데모를 다운로드하여 실행하십시오.


# Event 확장 설명
[Event 확장](https://php.net/manual/zh/book.event.php)은 필수적이지는 않지만, 업무가 1000보다 큰 동시 연결을 지원해야 할 때에는 Event 확장을 권장합니다. Event 확장을 설치하지 않아도 연결이 상대적으로 적으면(예: 1000개 미만의 동시 연결), 설치하지 않아도 됩니다.

## 일반적인 문제
1. `checking for include/event2/event.h... not found`와 같은 오류가 발생하면 우선 libevent-dev(el) 라이브러리를 삭제하고 libevent2-dev(el) 라이브러리를 설치해 보십시오.
CentOS 시스템: yum remove libevent-devel && yum install libevent2-devel
Debian/Ubuntu 시스템: apt-get remove libevent-dev && apt-get install libevent2-dev

2. `NOTICE: PHP message: PHP Warning: PHP Startup: Unable to load dynamic library '.../event.so' - ..../event.so: undefined symbol: php_sockets_le_socket in Unknown on line 0`와 같은 오류가 발생하면 event.so와 socket.so의 로드 순서를 변경하여 php.ini에 `extension=socket.so`를 `extension=event.so` 앞에 추가하여 socket 확장을 먼저 로드하도록 변경하십시오.
