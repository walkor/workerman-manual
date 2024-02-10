# Установка расширений
## Внимание
В отличие от режима работы с Apache+PHP или Nginx+PHP, WorkerMan работает на основе PHP командной строки [PHP CLI](http://php.net/manual/zh/features.commandline.php) с использованием другой исполняемой программы PHP и, возможно, отличающегося файла php.ini. Поэтому вывод команды ```phpinfo()``` в браузере, который указывает на установленное расширение, не означает, что в PHP CLI также установлено соответствующее расширение.

## Как узнать установленные расширения в PHP CLI
Запустите команду ```php -m```, чтобы вывести установленные расширения PHP CLI. Результат будет примерно следующим образом:
```shell
~# php -m
[PHP Modules]
event
posix
pcntl
...
```

## Как узнать местоположение файл php.ini для PHP CLI
Для того чтобы установить расширение, вам может понадобиться вручную настроить файл php.ini, добавив в него соответствующее расширение. Для этого необходимо знать местоположение файла php.ini для PHP CLI. Запустите ```php --ini``` для поиска файла php.ini для PHP CLI. Результат будет примерно следующим образом (у разных систем результат может отличаться):
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

# Установка расширений для PHP CLI (на примере установки расширения memcached)
## Метод 1: Использование команд apt или yum 
Если PHP устанавливался с помощью команд apt или yum, то расширение также можно установить с помощью apt или yum.

**Метод установки расширений для систем, таких как debian/ubuntu, с помощью apt (пользователям требуется использование команды sudo)**

1. Используйте команду ```apt-cache search``` для поиска пакета расширения
```shell
~# apt-cache search memcached php
php-apc - APC (Alternative PHP Cache) module for PHP 5
php5-memcached - memcached module for php5
```
2. Используйте команду ```apt-get install``` для установки пакета расширения
```shell
~# apt-get install -y php5-memcached
Reading package lists... Done
Reading state information... Done
...
```

**Метод установки расширений для систем, таких как centos, с помощью yum**

1. Используйте команду ```yum search``` для поиска пакета расширения
```shell
~# yum search memcached php
php-pecl-memcached - memcached module for php5
```
2. Используйте команду ```yum install``` для установки пакета расширения
```shell
~# yum install -y php-pecl-memcached
Reading package lists... Done
Reading state information... Done
...

**Примечание:**
Установка расширений с помощью apt или yum автоматически настраивает файл php.ini, что делает процесс установки очень удобным. Недостатком является то, что некоторые расширения могут отсутствовать в apt или yum.

## Метод 2: Установка с помощью pecl
Используйте команду ```pecl install``` для установки расширения

1. Установите с помощью ```pecl install```
```shell
~# pecl install memcached
downloading memcached-2.2.0.tgz ...
Starting to download memcached-2.2.0.tgz (70,449 bytes)
....
```
2. Настройте файл php.ini

Найдите местоположение файла php.ini, запустив ```php --ini``` и добавьте в него строку ```extension=memcached.so```

## Метод 3: Установка из исходного кода (чаще используется для установки встроенных в PHP расширений, как, например, pcntl)
1. Используйте команду ```php -v``` для просмотра версии вашего PHP CLI
```shell
~# php -v
PHP 5.3.29-1~dotdeb.0 with Suhosin-Patch (cli) (built: Aug 14 2014 19:55:20)
Copyright (c) 1997-2014 The PHP Group
Zend Engine v2.3.0, Copyright (c) 1998-2014 Zend Technologies
```
2. Скачайте исходный код PHP в соответствии с версией

Страница скачивания предыдущих версий PHP: https://php.net/releases/

3. Распакуйте архив с исходным кодом

Например, если имя загруженного архива - ```php-5.3.29.tar.gz```
```shell
~# tar -zxvf php-5.3.29.tar.gz
php-5.3.29/
php-5.3.29/README.WIN32-BUILD-SYSTEM
php-5.3.29/netware/
...
```
4. Перейдите в папку ext/pcntl исходного кода
```shell
~# cd php-5.3.29/ext/pcntl/
```
5. Выполните команду ```phpize```
```shell
~# phpize
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
```
6. Выполните команду ```configure```
```shell
~# ./configure
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
...
```
7. Выполните команду ```make```
```shell
~# make
/bin/bash /tmp/php-5.3.29/ext/pcntl/libtool --mode=compile cc ...
-I/usr/include/php5 -I/usr/include/php5/main -I/usr/include/php5/TSRM -I/usr/include/php5/Zend...
...
```
8. Выполните команду ```make install```
```shell
~# make install
Installing shared extensions:     /usr/lib/php5/20090626/
```
9. Настройте файл ini

Найдите местоположение файла php.ini, запустив ```php --ini``` и добавьте в него строку ```extension=pcntl.so```

**Примечание:**
Этот метод обычно используется для установки встроенных в PHP расширений, например, pcntl и posix. Кроме того, помимо компиляции конкретного расширения с помощью phpize, можно пересобрать весь PHP, добавив параметры для установки расширений, например:
```shell
~# ./configure --enable-pcntl --enable-posix ...
~# make && make install
```

## Метод 4: Установка с помощью phpize 
Если необходимое расширение отсутствует в каталоге ext исходного кода PHP, необходимо получить его с https://pecl.php.net и загрузить

Для установке расширения libevent (при условии наличия библиотеки libevent-dev)

1. Загрузите архив с расширением libevent (в любую папку на вашем сервере)
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

2. Распакуйте архив с расширением
```shell
~# tar -zxvf libevent-0.1.0.tgz
package.xml
libevent-0.1.0/config.m4
libevent-0.1.0/CREDITS
libevent-0.1.0/libevent.c
....
```

3. Перейдите в папку исходного кода
```shell
~# cd libevent-0.1.0/
```

4. Выполните команду ```phpize```
```shell
~# phpize
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
```

5. Выполните команду ```configure```
```shell
~# ./configure
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
checking for a sed that does not truncate output... /bin/sed
checking for cc... cc
checking whether the C compiler works... yes
...
```

6. Выполните команду ```make```
```shell
~# /bin/bash /data/test/libevent-0.1.0/libtool --mode=compile cc  -I. -I/data/test/libevent-0.1.0 -DPHP_ATOM_INC -I/data/test/libevent-0.1.0/include
...
```

7. Выполните команду ```make install```
```shell
~# make install
Installing shared extensions:     /usr/lib/php5/20090626/
```

8. Настройте файл ini

Найдите местоположение файла php.ini, запустив ```php --ini``` и добавьте в него строку ```extension=libevent.so```
