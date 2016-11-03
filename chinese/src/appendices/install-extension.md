# 安装扩展
## 注意
与Apache+PHP或者Nginx+PHP的运行模式不同，WorkerMan是基于命令行 [PHP Cli](http://php.net/manual/zh/features.commandline.php) 运行的，使用的是不同的PHP可执行程序，使用的php.ini文件也可能不同。所以在网页中打印```phpinfo()```看到安装了某个扩展，不代表命令行的PHP Cli也安装了对应的扩展。

## 如何确定PHP Cli安装了哪些扩展
运行 ```php -m``` 会列出命令行 PHP Cli 已经安装的扩展，结果类似如下：
```shell
~# php -m
[PHP Modules]
libevent
posix
pcntl
...
```

## 如何确定PHP Cli 的php.ini文件的位置
当我们安装扩展时，可能需要手动配置php.ini文件，把扩展加进去，所以要确认PHP Cli的php.ini文件的位置。可以运行```php --ini```查找PHP Cli的ini文件位置，结果类似如下(各个系统显示结果会有差异)：
```shell
~# php --ini
Configuration File (php.ini) Path: /etc/php5/cli
Loaded Configuration File:         /etc/php5/cli/php.ini
Scan for additional .ini files in: /etc/php5/cli/conf.d
Additional .ini files parsed:      /etc/php5/cli/conf.d/apc.ini,
/etc/php5/cli/conf.d/libevent.ini,
/etc/php5/cli/conf.d/memcached.ini,
/etc/php5/cli/conf.d/mysql.ini,
/etc/php5/cli/conf.d/pdo.ini,
/etc/php5/cli/conf.d/pdo_mysql.ini
...
```

# 给PHP Cli安装扩展（安装memcached扩展为例）
## 方法一、使用apt或者yum命令安装
如果PHP是通过 apt 或者 yum 命令安装的，则扩展也可以通过 apt 或者 yum 安装

**debian/ubuntu等系统apt安装PHP扩展方法（非root用户需要加sudo命令）**

1、利用```apt-cache search```查找扩展包
```shell
~# apt-cache search memcached php
php-apc - APC (Alternative PHP Cache) module for PHP 5
php5-memcached - memcached module for php5
```
2、使用```apt-get install```安装扩展包
```shell
~# apt-get install -y php5-memcached
Reading package lists... Done
Reading state information... Done
...
```

**centos等系统yum安装PHP扩展方法**

1、利用```yum search```查找扩展包
```shell
~# yum search memcached php
php-pecl-memcached - memcached module for php5
```
2、使用```yum install```安装扩展包
```shell
~# yum install -y php-pecl-memcached
Reading package lists... Done
Reading state information... Done
...
```
**说明：**

使用apt或者yum安装PHP扩展会自动配置php.ini文件，安装完直接可用，十分方便。缺点是有些扩展在apt或者yum中没有对应的扩展安装包。

## 方法二、使用pecl安装
使用```pecl install```命令安装扩展

1、```pecl install```安装
```
~# pecl install memcached
downloading memcached-2.2.0.tgz ...
Starting to download memcached-2.2.0.tgz (70,449 bytes)
....
```
2、配置php.ini

通过运行 ```php --ini```查找php.ini文件位置，然后在文件中添加```extension=memcached.so```

## 方法三、源码编译安装（一般是安装PHP自带的扩展，以安装pcntl扩展为例）
1、利用```php -v```命令查看当前的PHP Cli的版本
```shell
~# php -v
PHP 5.3.29-1~dotdeb.0 with Suhosin-Patch (cli) (built: Aug 14 2014 19:55:20)
Copyright (c) 1997-2014 The PHP Group
Zend Engine v2.3.0, Copyright (c) 1998-2014 Zend Technologies
```
2、根据版本下载PHP源代码

PHP历史版本下载页面：http://php.net/releases/

3、解压源码压缩包

例如下载的压缩包名称是```php-5.3.29.tar.gz```
```shell
~# tar -zxvf php-5.3.29.tar.gz
php-5.3.29/
php-5.3.29/README.WIN32-BUILD-SYSTEM
php-5.3.29/netware/
...
```
4、进入源码中的ext/pcntl目录
```shell
~# cd php-5.3.29/ext/pcntl/
```
5、运行 ```phpize``` 命令
```shell
~# phpize
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
```
6、运行 ```configure```命令
```shell
~# ./configure
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
...
```
7、运行 ```make``` 命令
```shell
~# make
/bin/bash /tmp/php-5.3.29/ext/pcntl/libtool --mode=compile cc ...
-I/usr/include/php5 -I/usr/include/php5/main -I/usr/include/php5/TSRM -I/usr/include/php5/Zend...
...
```
8、运 行```make install``` 命令
```shell
~# make install
Installing shared extensions:     /usr/lib/php5/20090626/
```
9、配置ini文件

通过运行 ```php --ini```查找php.ini文件位置，然后在文件中添加```extension=pcntl.so```

**说明：**
此方法一般用来安装PHP自带的扩展，例如posix扩展和pcntl扩展。除了用phpize编译某个扩展，也可以重新编译整个PHP，在编译时用参数添加扩展，例如在源码根目录运行
```shell
~# ./configure --enable-pcntl --enable-posix ...
~# make && make install
```

## 方法四、phpize安装
如果要安装的扩展在php源码ext目录中没有，那么这个扩展需要到http://pecl.php.net 搜索下载

以安装libevent扩展为例（假设系统安装了libevent-dev库）

1、下载libevent扩展文件压缩包（在当前系统哪个目录下载随意）
```shell
~# wget http://pecl.php.net/get/libevent-0.1.0.tgz
--2015-05-26 21:43:40--  http://pecl.php.net/get/libevent-0.1.0.tgz
Resolving pecl.php.net... 104.236.228.160
Connecting to pecl.php.net|104.236.228.160|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 9806 (9.6K) [application/octet-stream]
Saving to: “libevent-0.1.0.tgz”

100%[=======================================================>] 9,806       41.4K/s   in 0.2s

```

2、解压扩展文件压缩包
```shell
~# tar -zxvf libevent-0.1.0.tgz
package.xml
libevent-0.1.0/config.m4
libevent-0.1.0/CREDITS
libevent-0.1.0/libevent.c
....
```

3、进入到源码目录
```shell
~# cd libevent-0.1.0/
```

4、运行``` phpize ```命令
```shell
~# phpize
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
```

5、运行``` configure ```命令
```shell
~# ./configure
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
checking for a sed that does not truncate output... /bin/sed
checking for cc... cc
checking whether the C compiler works... yes
...
```

6、运行``` make ```命令
```shell
~# /bin/bash /data/test/libevent-0.1.0/libtool --mode=compile cc  -I. -I/data/test/libevent-0.1.0 -DPHP_ATOM_INC -I/data/test/libevent-0.1.0/include
...
```

7、运行``` make install ```命令
```shell
~# make install
Installing shared extensions:     /usr/lib/php5/20090626/
```

8、配置ini文件

通过运行 ```php --ini```查找php.ini文件位置，然后在文件中添加```extension=libevent.so```
