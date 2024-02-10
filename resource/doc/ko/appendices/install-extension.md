# 확장 설치

## 주의
Apache+PHP 또는 Nginx+PHP와는 다르게 WorkerMan은 PHP 커맨드 라인 [PHP CLI](http://php.net/manual/zh/features.commandline.php)을 기반으로 하며, 다른 PHP 실행 파일을 사용하며 php.ini 파일도 다를 수 있습니다. 따라서 웹페이지에서 ```phpinfo()```를 통해 특정 확장이 설치되었다고 하더라도, 커맨드 라인의 PHP CLI에도 해당 확장이 설치되어 있는 것은 아닙니다.

## PHP CLI에 설치된 확장 확인하는 방법
```php -m``` 명령어를 실행하여 커맨드 라인의 PHP CLI에 설치된 확장을 나열할 수 있으며, 결과는 다음과 같습니다:
```shell
~# php -m
[PHP Modules]
event
posix
pcntl
...
```

## PHP CLI의 php.ini 파일 위치 확인하는 방법
확장을 설치하는 중에는 수동적으로 php.ini 파일을 구성하여 해당 확장을 추가해야 할 수 있으므로 PHP CLI의 php.ini 파일 위치를 확인해야 합니다. ```php --ini```를 실행하여 PHP CLI의 ini 파일 위치를 찾을 수 있으며, 결과는 다음과 같습니다(다양한 시스템에서 결과가 다를 수 있음):
```shell
~# php --ini
Configuration File (php.ini) Path: /etc/php8/cli
Loaded Configuration File:         /etc/php8/cli/php.ini
Scan for additional .ini files in: /etc/php8/cli/conf.d
Additional .ini files parsed:      /etc/php8/cli/conf.d/apc.ini,
/etc/php8/cli/conf.d/pdo.ini,
/etc/php8/cli/conf.d/pdo_mysql.ini
...
```

# PHP CLI에 확장 설치하기 (memcached 확장을 설치하는 예)
## 방법 1: apt 또는 yum 명령어를 사용하여 설치
PHP가 apt 또는 yum 명령어를 통해 설치된 경우, 확장 또한 apt 또는 yum을 통해 설치할 수 있습니다

**debian/ubuntu 등 시스템에서 apt를 통한 PHP 확장 설치 방법 (일반 사용자는 sudo 명령어를 추가해야 함)**
1. ```apt-cache search```를 사용하여 확장 패키지를 찾습니다.
```shell
~# apt-cache search memcached php
php-apc - APC (Alternative PHP Cache) module for PHP 5
php5-memcached - memcached module for php5
```
2. ```apt-get install```을 사용하여 확장 패키지를 설치합니다.
```shell
~# apt-get install -y php5-memcached
Reading package lists... Done
Reading state information... Done
...
```

**centos 등 시스템에서 yum을 통한 PHP 확장 설치 방법**
1. ```yum search```를 사용하여 확장 패키지를 찾습니다.
```shell
~# yum search memcached php
php-pecl-memcached - memcached module for php5
```
2. ```yum install```을 사용하여 확장 패키지를 설치합니다.
```shell
~# yum install -y php-pecl-memcached
Reading package lists... Done
Reading state information... Done
...
```
**참고:**
apt 또는 yum을 통해 PHP 확장을 설치하면 php.ini 파일이 자동으로 설정되어 설치 후 즉시 사용할 수 있습니다. 단점으로는 일부 확장은 apt 또는 yum에 해당하는 확장 패키지가 없을 수 있습니다.

## 방법 2: pecl을 통한 설치
```pecl install``` 명령어를 사용하여 확장을 설치합니다.

1. ```pecl install```로 설치합니다.
```shell
~# pecl install memcached
downloading memcached-2.2.0.tgz ...
Starting to download memcached-2.2.0.tgz (70,449 bytes)
....
```
2. php.ini를 구성합니다.
```php --ini```를 실행하여 php.ini 파일의 위치를 찾은 후 해당 파일에 ```extension=memcached.so```를 추가합니다.

## 방법 3: 소스 코드 컴파일 설치 (일반적으로 PHP 내장 확장을 설치할 때 사용, pcntl 확장 설치를 예로 들음)
1. ```php -v``` 명령어를 사용하여 현재 PHP CLI의 버전을 확인합니다.
```shell
~# php -v
PHP 5.3.29-1~dotdeb.0 with Suhosin-Patch (cli) (built: Aug 14 2014 19:55:20)
Copyright (c) 1997-2014 The PHP Group
Zend Engine v2.3.0, Copyright (c) 1998-2014 Zend Technologies
```
2. 해당 버전에 맞는 PHP 소스 코드를 다운로드합니다.

PHP 이전 버전 다운로드 페이지: https://php.net/releases/

3. 소스 코드 압축을 해제합니다.
예를 들어, 다운로드한 압축 파일 이름이 ```php-5.3.29.tar.gz```라면 다음과 같이 작업합니다.
```shell
~# tar -zxvf php-5.3.29.tar.gz
php-5.3.29/
php-5.3.29/README.WIN32-BUILD-SYSTEM
php-5.3.29/netware/
...
```
4. 소스 코드의 ext/pcntl 디렉토리로 들어갑니다.
```shell
~# cd php-5.3.29/ext/pcntl/
```
5. ```phpize``` 명령어를 실행합니다.
```shell
~# phpize
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
```
6. ```configure``` 명령어를 실행합니다.
```shell
~# ./configure
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
...
```
7. ```make``` 명령어를 실행합니다.
```shell
~# make
/bin/bash /tmp/php-5.3.29/ext/pcntl/libtool --mode=compile cc ...
-I/usr/include/php5 -I/usr/include/php5/main -I/usr/include/php5/TSRM -I/usr/include/php5/Zend...
...
```
8. ```make install``` 명령어를 실행합니다.
```shell
~# make install
Installing shared extensions:     /usr/lib/php5/20090626/
```
9. ini 파일을 설정합니다.
```php --ini```를 실행하여 php.ini 파일의 위치를 찾은 후 해당 파일에 ```extension=pcntl.so```를 추가합니다.

**참고:**
이 방법은 일반적으로 PHP 내장 확장을 설치할 때 사용되며, 예를 들어 posix 확장 및 pcntl 확장 등이 있습니다. 확장을 phpize로 컴파일하는 대신 PHP 전체를 다시 컴파일하여 확장을 추가하는 매개변수로 컴파일할 수도 있습니다. 예를 들어, 소스 코드 루트 디렉토리에서 다음과 같이 작업할 수 있습니다.
```shell
~# ./configure --enable-pcntl --enable-posix ...
~# make && make install
```

## 방법 4: phpize를 통한 설치
설치하려는 확장이 PHP 소스 코드의 ext 디렉토리에 없는 경우, 해당 확장을 https://pecl.php.net에서 검색 및 다운로드해야 합니다.

libevent 확장을 설치하는 예를 들겠습니다 (시스템에 libevent-dev 라이브러리가 설치되어 있다고 가정)

1. libevent 확장 파일 압축을 다운로드받습니다 (현재 시스템의 어느 디렉토리에서 다운로드되어도 상관 없음)
```shell
~# wget https://pecl.php.net/get/libevent-0.1.0.tgz
--2015-05-26 21:43:40--  https://pecl.php.net/get/libevent-0.1.0.tgz
Resolving pecl.php.net... 104.236.228.160
Connecting to pecl.php.net|104.236.228.160|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 9806 (9.6K) [application/octet-stream]
Saving to: “libevent-0.1.0.tgz”

100%[=======================================================>] 9,806       41.4K/s   in 0.2s

```

2. 확장 파일 압축을 해제합니다.
```shell
~# tar -zxvf libevent-0.1.0.tgz
package.xml
libevent-0.1.0/config.m4
libevent-0.1.0/CREDITS
libevent-0.1.0/libevent.c
....
```

3. 소스 코드 디렉토리로 들어갑니다.
```shell
~# cd libevent-0.1.0/
```

4. ```phpize``` 명령어를 실행합니다.
```shell
~# phpize
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
```

5. ```configure``` 명령어를 실행합니다.
```shell
~# ./configure
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
checking for a sed that does not truncate output... /bin/sed
checking for cc... cc
checking whether the C compiler works... yes
...
```

6. ```make``` 명령어를 실행합니다.
```shell
~# /bin/bash /data/test/libevent-0.1.0/libtool --mode=compile cc  -I. -I/data/test/libevent-0.1.0 -DPHP_ATOM_INC -I/data/test/libevent-0.1.0/include
...
```

7. ```make install``` 명령어를 실행합니다.
```shell
~# make install
Installing shared extensions:     /usr/lib/php5/20090626/
```

8. ini 파일을 설정합니다.
```php --ini```를 실행하여 php.ini 파일의 위치를 찾은 후 해당 파일에 ```extension=libevent.so```를 추가합니다.
