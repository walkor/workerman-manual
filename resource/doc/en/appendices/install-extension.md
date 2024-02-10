# Extension Installation
## Note
Unlike the running mode of Apache+PHP or Nginx+PHP, Workerman runs based on PHP command line [PHP CLI](http://php.net/manual/zh/features.commandline.php), using a different PHP executable and possibly a different php.ini file. Therefore, seeing that a certain extension is installed when using `phpinfo()` in a web page does not mean that the corresponding extension is also installed for the PHP CLI.

## How to Determine Which Extensions are Installed for PHP CLI
Running ```php -m``` will list the extensions already installed for the command line PHP CLI, the result is similar to the following:
```shell
~# php -m
[PHP Modules]
event
posix
pcntl
...
```

## How to Determine the Location of the php.ini file for PHP CLI
When installing extensions, it may be necessary to manually configure the php.ini file to add the extensions, so it's important to confirm the location of the php.ini file for PHP CLI. This can be done by running ```php --ini``` to find the location of the PHP CLI ini file, the result is similar to the following (results may vary for different systems):
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

# Installing Extensions for PHP CLI (Installing the memcached extension as an example)
## Method One: Using apt or yum to Install
If PHP was installed using apt or yum commands, the extensions can also be installed using apt or yum.

**Method for installing PHP extensions on debian/ubuntu and similar systems using apt (non-root users need to add sudo command)**
1. Use ```apt-cache search``` to find the extension package
```shell
~# apt-cache search memcached php
php-apc - APC (Alternative PHP Cache) module for PHP 5
php5-memcached - memcached module for php5
```
2. Use ```apt-get install``` to install the extension package
```shell
~# apt-get install -y php5-memcached
Reading package lists... Done
Reading state information... Done
...
```

**Method for installing PHP extensions on centos and similar systems using yum**
1. Use ```yum search``` to find the extension package
```shell
~# yum search memcached php
php-pecl-memcached - memcached module for php5
```
2. Use ```yum install``` to install the extension package
```shell
~# yum install -y php-pecl-memcached
Reading package lists... Done
Reading state information... Done
...
```

**Note:**
Using apt or yum to install PHP extensions will automatically configure the php.ini file, making the extension immediately usable. However, some extensions may not have corresponding installation packages in apt or yum.

## Method Two: Using pecl to Install
Use the ```pecl install``` command to install an extension

1. Install with ```pecl install```
```shell
~# pecl install memcached
downloading memcached-2.2.0.tgz ...
Starting to download memcached-2.2.0.tgz (70,449 bytes)
....
```
2. Configure php.ini

Find the location of the php.ini file by running ```php --ini```, then add ```extension=memcached.so``` to the file.

## Method Three: Compiling and Installing from Source Code (Installing the pcntl extension as an example)
1. Use the ```php -v``` command to check the current version of PHP CLI
```shell
~# php -v
PHP 5.3.29-1~dotdeb.0 with Suhosin-Patch (cli) (built: Aug 14 2014 19:55:20)
Copyright (c) 1997-2014 The PHP Group
Zend Engine v2.3.0, Copyright (c) 1998-2014 Zend Technologies
```
2. Download PHP source code based on the version

PHP historical version download page: https://php.net/releases/

3. Extract the source code archive

For example, if the downloaded archive is named ```php-5.3.29.tar.gz```
```shell
~# tar -zxvf php-5.3.29.tar.gz
php-5.3.29/
php-5.3.29/README.WIN32-BUILD-SYSTEM
php-5.3.29/netware/
...
```
4. Navigate to the ext/pcntl directory in the source code
```shell
~# cd php-5.3.29/ext/pcntl/
```
5. Run the ```phpize``` command
```shell
~# phpize
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
```
6. Run the ```configure``` command
```shell
~# ./configure
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
...
```
7. Run the ```make``` command
```shell
~# make
/bin/bash /tmp/php-5.3.29/ext/pcntl/libtool --mode=compile cc ...
-I/usr/include/php5 -I/usr/include/php5/main -I/usr/include/php5/TSRM -I/usr/include/php5/Zend...
...
```
8. Run ```make install```
```shell
~# make install
Installing shared extensions:     /usr/lib/php5/20090626/
```
9. Configure the ini file

Find the location of the php.ini file by running ```php --ini```, then add ```extension=pcntl.so``` to the file.

**Note:**
This method is generally used to install built-in PHP extensions, such as posix and pcntl. In addition to compiling a specific extension with phpize, it is also possible to recompile the entire PHP, adding extensions with parameters during compilation, for example, running
```shell
~# ./configure --enable-pcntl --enable-posix ...
~# make && make install
```

## Method Four: Installing with phpize
If the extension to be installed is not in the ext directory of the PHP source code, it needs to be searched and downloaded from https://pecl.php.net

As an example, installing the libevent extension (assuming that the system has the libevent-dev library installed)

1. Download the libevent extension file archive (download it to any directory on the system)
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

2. Extract the extension file archive
```shell
~# tar -zxvf libevent-0.1.0.tgz
package.xml
libevent-0.1.0/config.m4
libevent-0.1.0/CREDITS
libevent-0.1.0/libevent.c
....
```

3. Navigate to the source code directory
```shell
~# cd libevent-0.1.0/
```

4. Run ```phpize```
```shell
~# phpize
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
```

5. Run ```configure```
```shell
~# ./configure
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
checking for a sed that does not truncate output... /bin/sed
checking for cc... cc
checking whether the C compiler works... yes
...
```

6. Run ```make```
```shell
~# /bin/bash /data/test/libevent-0.1.0/libtool --mode=compile cc  -I. -I/data/test/libevent-0.1.0 -DPHP_ATOM_INC -I/data/test/libevent-0.1.0/include
...
```

7. Run ```make install```
```shell
~# make install
Installing shared extensions:     /usr/lib/php5/20090626/
```

8. Configure the ini file

Find the location of the php.ini file by running ```php --ini```, then add ```extension=libevent.so``` to the file.
